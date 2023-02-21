# Speedb Use Cases

Speedb is an embedded key-value store that stores data as key-value pairs and is designed to be embedded in applications. Speedb is commonly used in a variety of applications where low latency and high performance are important, such as in web browsers, streaming applications and mobile apps.



Speedb is primarily used for the following use cases:\


1. **Streaming and microprocessing:** increasing application performance such as streaming applications, microprocessing and more. Streaming applications like Apache Flink and Kafka Streams are using Speedb to store states in a fast and efficient manner.&#x20;
2. **Storage engine:** A storage engine usually handles basic operations of storage management, most notably to create, read, update, and delete (CRUD) data. In addition, the data engine needs to efficiently provide an interface for sequential read of data and atomic update of several keys at the same time. Replacing the existing storage engine of a database with Speedb will boost the performance and ensure consistent performance over time.&#x20;
3. **Real-time data processing by IOT**: Internet of Things devices oftenly collect and generate large amounts of data, which needs to be stored for future use. Also it sometimes needs processing power to perform complex data processing tasks. Speedb is used as a high performance storage engine to perform these tasks and store the large amount of data that is needed.&#x20;
4. **Storage Metadata**: As the amount of data and number of files (and object) is increasing dramatically over time, the amount of metadata is growing out of control.\
   Since metadata needs to be accessed fast, on a large scale, and needs to be consistent, an embedded KVS is the preferred method of metadata management from most storage systems today.
5. **Gaming:** store game state data, allowing for fast access and updates to game data such as state of a game, including player progress, game statistics, user profile and preferences. The fast access and the high performance of Speedb makes it well suited in games that need to store and retrieve large amounts of data quickly.&#x20;
6. **Mobile apps:** store data locally on mobile devices, allowing for offline access and faster data retrieval. Using Speedb on mobile applications can help to improve the performance, user experience and overall functionality of the application.&#x20;
7. **Caching**: Speedb can be used as a cache to store frequently accessed data, allowing for faster access and reduced load on the underlying database.
8. **Metrics and analytics:** Speedb can  be used to store and retrieve metrics and analytics data, such as application performance data or user behavior data. This can help to provide valuable insights into the usage of the app and inform decisions about future development.

\
\
\
\
\
