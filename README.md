asyncapi: '2.6.0'
id: 'urn:my_async_producer_app:server'
info:
  title: Async API specification for Kafka
  version: '1.0.0'
  description: 'Спецификация асинхронного обмена данными через Kafka'

servers:
  dev:
    url: my-kafka.upstash.io:9092
    protocol: kafka-secure
    description: 'параметры брокера Kafka'
    security:
      - saslScram: [SCRAM-SHA-256]

defaultContentType: application/json

channels:
  publishing.{eventId}:
    description: приложение-продюсер
    parameters:
      eventId:
        $ref: '#/components/parameters/eventId'
    publish:
      summary: 'Генерация события, отправляемого с клиентского приложения'
      operationId: PublishNewEvent
      traits:
        - $ref: '#/components/operationTraits/kafka'
      message:
           $ref: '#/components/messages/booking'
    bindings:
      kafka:
  
  bookingTopic:
    description: 'раздел для вопросов'
    subscribe:
      operationId: bookingTopic_performing
      traits:
        - $ref: '#/components/operationTraits/kafka'
      message:
        $ref: '#/components/messages/booking'

components:
  messages:
    booking:
      name: booking
      title: booking
      summary: 'в раздел 0 попадают все вопросы'
      contentType: application/json
      payload:
        $ref: "#/components/schemas/booking"   

  schemas:
    booking:
      type: object
      description: 'заявка на бронирование отеля'
      properties:
        id:
          type: string
          description: 'q-x-xx-xx-xxxx'
          example: 8-8-77-89-5412
        date:
          type: string
          format: date
          description: 'дата формирования заявки'
          example: 2023-09-30
        state:
          type: string
          enum:
            - принят
            - подтвержден
            - отменен
          description: 'статус'
          example: принят
        dateFrom:
          type: string
          format: date
          description: 'дата начала брони'
          example: 2023-10-01
        dateBy:
          type: string
          format: date
          description: 'дата окончания брони'
          example: 2023-10-25
        paymentMethod:
          type: string
          enum:
            - банковская карта
            - банковский перевод
            - наличные
          description: 'способы оплаты'  
          example: наличные 
        amount:
          type: number
          description: 'сумма бронирования' 
          example: 10000
        hotel:
          type: string
          description: 'отель'
          example: Hilton
        city:
          type: string
          description: 'город'
          example: Москва

      
  securitySchemes:
    saslScram:
      type: scramSha256
      description: протокол безопасности SASL_SSL, механизм безопасности SCRAM-SHA-256

  parameters:
    eventId:
      description: идентификатор события
      schema:
        type: string
        description: 'x-xx:xx-xx-xx-xxx'

  operationTraits:
    kafka:
      bindings:
        kafka:
          topic: 'inputs_topic'
          partitions: 1
          replicas: 1
          topicConfiguration:
          cleanup.policy: ["delete", "compact"]
          retention.ms: 604800000
          retention.bytes: 1000000000
          delete.retention.ms: 86400000
          max.message.bytes: 1048588
        bindingVersion: '0.4.0'
