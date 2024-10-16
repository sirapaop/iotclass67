# Analyze and make aggregations.
ในการวิเคราะห์ข้อมูลจากหัวข้อ iot-frames ที่รับข้อมูลจากเซนเซอร์ เราจะนำข้อมูลไปประมวลผลเพื่อให้ได้ข้อมูลที่มีประโยชน์ โดยในสถาปัตยกรรมนี้เราใช้ microservice ที่ชื่อว่า iot-processor ซึ่งใช้งาน Kafka Streams และมีการตั้งค่าทั้งหมดภายในคลาสนี้สำหรับการประมวลผลสตรีมข้อมูล
```java
@Configuration
@EnableKafka
@EnableKafkaStreams
public class KafkaStreamsConfig {

    @Value("${kafka.topic.input}")
    private String inputTopic;

    @Autowired
    private AggregateMetricsBySensorProcessor aggregateMetricsBySensorProcessor;

    @Autowired
    private AggregateMetricsByPlaceProcessor aggregateMetricsByPlaceProcessor;

    @Autowired
    private MetricsTimeSeriesProcessor metricsTimeSeriesProcessor;

    @Bean(name = KafkaStreamsDefaultConfiguration.DEFAULT_STREAMS_CONFIG_BEAN_NAME)
    public KafkaStreamsConfiguration provideKStreamsConfigs(final KafkaProperties kafkaProperties) {
        Map<String, Object> config = new HashMap<>();
        config.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, kafkaProperties.getBootstrapServers());
        config.put(StreamsConfig.APPLICATION_ID_CONFIG, kafkaProperties.getClientId());
        config.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, SensorKeySerde.class.getName());
        config.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, SensorDataSerde.class.getName());
        return new KafkaStreamsConfiguration(config);
    }

    @Bean
    public KStream<SensorKeyDTO, SensorDataDTO> provideKStream(StreamsBuilder kStreamBuilder) {
        final KStream<SensorKeyDTO, SensorDataDTO> stream = kStreamBuilder.stream(inputTopic);
        aggregateMetricsBySensorProcessor.process(stream);
        aggregateMetricsByPlaceProcessor.process(stream);
        metricsTimeSeriesProcessor.process(stream);
        return stream;
    }
}
```

## การรวมข้อมูลตามเซนเซอร์
ตัวประมวลผลตัวแรก Aggregate Metrics By Sensor Processor จะทำการรวมข้อมูลตาม sensor id โดยใช้ rotating time window ทุกๆ 5 นาทีเพื่อคำนวณค่าเฉลี่ย เช่น อุณหภูมิ ความชื้น ความดัน แสงสว่าง โดยข้อมูลที่รวมนี้จะถูกส่งไปยัง Kafka โดยใช้ SerDe เพื่อทำการ serialize และ deserialize ข้อมูล

```java
@Component
public class AggregateMetricsBySensorProcessor {

    private final static int WINDOW_SIZE_IN_MINUTES = 5;

    @Value("${kafka.topic.aggregate-metrics-sensor}")
    private String aggMetricsSensorOutput;

    public void process(KStream<SensorKeyDTO, SensorDataDTO> stream) {
        buildAggregateMetricsBySensor(stream)
                .to(aggMetricsSensorOutput, Produced.with(String(), new SensorAggregateMetricsSensorSerde()));
    }

    private KStream<String, SensorAggregateSensorMetricsDTO> buildAggregateMetricsBySensor(KStream<SensorKeyDTO, SensorDataDTO> stream) {
        return stream
                .map((key, val) -> new KeyValue<>(val.getId(), val))
                .groupByKey(Grouped.with(String(), new SensorDataSerde()))
                .windowedBy(TimeWindows.of(Duration.ofMinutes(WINDOW_SIZE_IN_MINUTES)))
                .aggregate(SensorAggregateSensorMetricsDTO::new,
                        (String k, SensorDataDTO v, SensorAggregateSensorMetricsDTO va) -> aggregateData(v, va),
                         buildWindowPersistentStore()
                )
                .suppress(Suppressed.untilWindowCloses(unbounded()))
                .toStream()
                .map((key, value) -> KeyValue.pair(key.key(), value));
    }

    private Materialized<String, SensorAggregateSensorMetricsDTO, WindowStore<Bytes, byte[]>> buildWindowPersistentStore() {
        return Materialized
                .<String, SensorAggregateSensorMetricsDTO, WindowStore<Bytes, byte[]>>as("aggregate-metrics-by-sensor-tmp")
                .withKeySerde(String())
                .withValueSerde(new SensorAggregateMetricsSensorSerde());
    }

    private SensorAggregateSensorMetricsDTO aggregateData(final SensorDataDTO v, final SensorAggregateSensorMetricsDTO va) {
        va.setId(v.getId());
        va.setName(v.getName());
        va.setCountMeasures(va.getCountMeasures() + 1);
        va.setSumTemperature(va.getSumTemperature() + v.getPayload().getTemperature());
        va.setAvgTemperature(va.getSumTemperature() / va.getCountMeasures());
        va.setSumHumidity(va.getSumHumidity() + v.getPayload().getHumidity());
        va.setAvgHumidity(va.getSumHumidity() / va.getCountMeasures());
        va.setSumLuminosity(va.getSumLuminosity() + v.getPayload().getLuminosity());
        va.setAvgLuminosity(va.getSumLuminosity() / va.getCountMeasures());
        va.setSumPressure(va.getSumPressure() + v.getPayload().getPressure());
        va.setAvgPressure(va.getSumPressure() / va.getCountMeasures());
        return va;
    }
}
```
เมื่อ rotating time window ปิด ข้อมูลการประมวลผลจะถูกสร้างเป็นบันทึกสำหรับแต่ละเซนเซอร์ และข้อมูลนี้จะถูกเก็บในหัวข้อ iot-aggregate-metrics-by-sensor

## การรวมข้อมูลตามสถานที่
ตัวประมวลผลที่สอง Aggregate Metrics By Place Processor ทำการรวมข้อมูลแบบเดียวกับเซนเซอร์ แต่ใช้ตัวระบุสถานที่เพื่อคำนวณค่าเฉลี่ยตามสถานที่

```java
@Component
public class AggregateMetricsByPlaceProcessor {

    private final static int WINDOW_SIZE_IN_MINUTES = 5;

    @Value("${kafka.topic.aggregate-metrics-place}")
    private String aggMetricsPlaceOutput;

    public void process(KStream<SensorKeyDTO, SensorDataDTO> stream) {
        buildAggregateMetrics(stream)
                .to(aggMetricsPlaceOutput, Produced.with(String(), new SensorAggregateMetricsPlaceSerde()));
    }

    private KStream<String, SensorAggregatePlaceMetricsDTO> buildAggregateMetrics(KStream<SensorKeyDTO, SensorDataDTO> stream) {
        return stream
                .map((key, val) -> new KeyValue<>(val.getPlaceId(), val))
                .groupByKey(Grouped.with(String(), new SensorDataSerde()))
                .windowedBy(TimeWindows.of(Duration.ofMinutes(WINDOW_SIZE_IN_MINUTES)))
                .aggregate(SensorAggregatePlaceMetricsDTO::new,
                        (String k, SensorDataDTO v, SensorAggregatePlaceMetricsDTO va) -> aggregateData(v, va),
                        buildWindowPersistentStore()
                )
                .suppress(Suppressed.untilWindowCloses(unbounded()))
                .toStream()
                .map((key, value) -> KeyValue.pair(key.key(), value));
    }

    private Materialized<String, SensorAggregatePlaceMetricsDTO, WindowStore<Bytes, byte[]>> buildWindowPersistentStore() {
        return Materialized
                .<String, SensorAggregatePlaceMetricsDTO, WindowStore<Bytes, byte[]>>as("aggregate-metrics-by-place-tmp")
                .withKeySerde(String())
                .withValueSerde(new SensorAggregateMetricsPlaceSerde());
    }

    private SensorAggregatePlaceMetricsDTO aggregateData(final SensorDataDTO v, final SensorAggregatePlaceMetricsDTO va) {
        va.setPlaceId(v.getId());
        va.setCountMeasures(va.getCountMeasures() + 1);
        va.setSumTemperature(va.getSumTemperature() + v.getPayload().getTemperature());
        va.setAvgTemperature(va.getSumTemperature() / va.getCountMeasures());
        va.setSumHumidity(va.getSumHumidity() + v.getPayload().getHumidity());
        va.setAvgHumidity(va.getSumHumidity() / va.getCountMeasures());
        va.setSumLuminosity(va.getSumLuminosity() + v.getPayload().getLuminosity());
        va.setAvgLuminosity(va.getSumLuminosity() / va.getCountMeasures());
        va.setSumPressure(va.getSumPressure() + v.getPayload().getPressure());
        va.setAvgPressure(va.getSumPressure() / va.getCountMeasures());
        return va;
    }
}
```

## การแปลงข้อมูลเป็นรูปแบบที่เข้ากับ Prometheus
โปรเซสเซอร์ตัวสุดท้าย Metrics Time Series Processor จะเปลี่ยนข้อมูลเป็น schema ที่รองรับการทำงานกับ Prometheus โดยใช้ sensor id และ place id เป็นมิติข้อมูล และ Payload จะถูกแสดงเป็นข้อมูลที่สามารถใช้ใน Grafana
```java
@Component
public class MetricsTimeSeriesProcessor {

    @Value("${kafka.topic.metrics-time-series}")
    private String metricTimeSeriesOutput;

    public void process(KStream<SensorKeyDTO, SensorDataDTO> stream) {
        stream
                .map((key, val) -> new KeyValue<>(val.getId(), buildSensorTimeSerieMetric(val)))
                .to(metricTimeSeriesOutput, Produced.with(String(), new SensorTimeSerieMetricSerde()));
    }

    private SensorTimeSerieMetricDTO buildSensorTimeSerieMetric(final SensorDataDTO sensorData) {
        return SensorTimeSerieMetricDTO.builder()
                .name("sample_sensor_metric")
                .timestamp(new Date().getTime())
                .type("sensor")
                .dimensions(SensorTimeSerieMetricDimensionsDTO.builder()
                        .placeId(sensorData.getPlaceId())
                        .sensorId(sensorData.getId())
                        .sensorName(sensorData.getName())
                        .build())
                .values(SensorTimeSerieMetricValuesDTO.builder()
                        .humidity((double) sensorData.getPayload().getHumidity())
                        .luminosity((double) sensorData.getPayload().
```