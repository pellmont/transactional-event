# Transactional Event Library for Jakarta EE 8

A [Transactional Event Library](https://jonasrutishauser.github.io/transactional-event/) that implements the [outbox pattern](https://microservices.io/patterns/data/transactional-outbox.html) for Jakarta EE 8.

[![GNU Lesser General Public License, Version 3, 29 June 2007](https://img.shields.io/github/license/jonasrutishauser/transactional-event.svg?label=License)](http://www.gnu.org/licenses/lgpl-3.0.txt)
[![Maven Central](https://img.shields.io/maven-central/v/com.github.jonasrutishauser/transactional-event-api.svg?label=Maven%20Central)](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.github.jonasrutishauser%22%20a%3A%22transactional-event-api%22)
[![Build Status](https://img.shields.io/github/workflow/status/jonasrutishauser/transactional-event/Maven%20CI.svg?label=Build)](https://github.com/jonasrutishauser/transactional-event/actions)
[![Coverage](https://img.shields.io/codecov/c/github/jonasrutishauser/transactional-event/master.svg?label=Coverage)](https://codecov.io/gh/jonasrutishauser/transactional-event)

## Used Jakarta EE APIs
The following APIs are required:
- CDI 2.0
- Concurrency Utilities 1.0
- JDBC 4.2
- JTA 1.2

The following APIs are optionally supported for serialization:
- JAXB 2.2
- JSON-B 1.0

## Publish an Event
An Event can be published using the [`EventPublisher`](https://jonasrutishauser.github.io/transactional-event/snapshot/transactional-event-api/apidocs/?com/github/jonasrutishauser/transactional/event/api/EventPublisher.html) API:

```java
   @Inject
   private EventPublisher publisher;
   
   public void someMethod() {
      ...
      SomeEvent event = ...
      publisher.publish(event);
      ...
   }
```

## Handle an Event
For every event type published there must be a corresponding [`Handler`](https://jonasrutishauser.github.io/transactional-event/snapshot/transactional-event-api/apidocs/?com/github/jonasrutishauser/transactional/event/api/handler/Handler.html) (qualified by [`EventHandler`](https://jonasrutishauser.github.io/transactional-event/snapshot/transactional-event-api/apidocs/?com/github/jonasrutishauser/transactional/event/api/handler/EventHandler.html)):

```java
@Dependent
@EventHandler
class SomeEventHandler extends AbstractHandler<SomeEvent> {
   @Override
   protected void handle(SomeEvent event) {
      ...
   }
}
```

## Data Source
The library expects that the following table exists when using the `javax.sql.DataSource` with the [`Events`](https://jonasrutishauser.github.io/transactional-event/snapshot/transactional-event-api/apidocs/?com/github/jonasrutishauser/transactional/event/api/Events.html) qualifier:

```sql
CREATE TABLE event_store (
	id VARCHAR(50) NOT NULL,
	event_type VARCHAR(50) NOT NULL,
	payload VARCHAR(4000) NOT NULL,
	published_at TIMESTAMP NOT NULL,
	tries INT NOT NULL,
	lock_owner VARCHAR(50),
	locked_until BIGINT NOT NULL,
	PRIMARY KEY (id)
);

CREATE INDEX event_store_locked_until ON event_store (locked_until);
```

The required `javax.sql.DataSource` can be specified like the following:

```java
@Dependent
class EventsDataSource {
   @Events
   @Produces
   @Resource(name = "someDb")
   private DataSource dataSource;
}
```