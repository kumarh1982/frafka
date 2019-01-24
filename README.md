# Frafka

Frafka is a Kafka implementation for [Frizzle](github.com/qntfy/frizzle) based on [confluent-go-kafka](https://github.com/confluentinc/confluent-kafka-go).

Frizzle is a magic message (`Msg`) bus designed for parallel processing w many goroutines.
  * `Receive()` messages from a configured `Source`
  * Do your processing, possibly `Send()` each `Msg` on to one or more `Sink` destinations
  * `Ack()` (or `Fail()`) the `Msg`  to notify the `Source` that processing completed

## Prereqs / Build instructions

### Go mod

As of Go 1.11, frafka uses [go mod](https://github.com/golang/go/wiki/Modules) for dependency management.

### Install librdkafka

Frafka depends on C library `librdkafka` (>=`v0.11.4`). For Debian 9+ (which includes golang docker images),
it has to be built from source. Fortunately, there's a script for that.
```
  # Install librdkafka
  - curl --silent -OL https://raw.githubusercontent.com/confluentinc/confluent-kafka-go/v0.11.4/mk/bootstrap-librdkafka.sh
  - bash bootstrap-librdkafka.sh v0.11.4 /usr/local
  - ldconfig
```

Once that is installed, should be good to go with
```
$ go get github.com/qntfy/frafka
$ cd frafka
$ go build
```

## Running the tests

`go test -v --cover ./...`

## Configuration
Frafka Sources and Sinks are configured using [Viper](https://godoc.org/github.com/spf13/viper).
```
func InitSink(config *viper.Viper) (*Sink, error)

func InitSource(config *viper.Viper) (*Source, error)
```

We typically initialize Viper through environment variables (but client can do whatever it wants,
just needs to provide the configured Viper object with relevant values). The application might
use a prefix before the below values.

| Variable | Required | Description | Default |
|---------------------------|:--------:|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:-------:|
| KAFKA_BROKERS | required | address(es) of kafka brokers, space separated |  |
| KAFKA_TOPICS | source | topic(s) to read from |  |
| KAFKA_CONSUMER_GROUP | source | consumer group value for coordinating multiple clients |  |
| KAFKA_CONSUME_LATEST_FIRST | source (optional) | start at the beginning or end of topic | earliest |

## Async Error Handling
Since records are sent in batch fashion, Kafka may report errors or other information asynchronously.
Event can be recovered via channels returned by the `Sink.Events()` and `Source.Events()` methods.
Partition changes and EOF will be reported as non-error Events, other errors will conform to `error` interface.
Where possible, Events will retain underlying type from [confluent-kafka-go](https://github.com/confluentinc/confluent-kafka-go)
if more information is desired.

## Contributing
Contributions welcome! Take a look at open issues.
