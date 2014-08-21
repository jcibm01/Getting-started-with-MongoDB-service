Getting-started-with-MongoDB-service
====================================

Getting started with MongoDB service

Tomado de https://www.ng.bluemix.net/docs/#services/MongoDB/index.html#bind_mongo


This community service is as-is and is provided for development and experimentation purposes only.
MongoDB is an open source document, NoSQL database that is owned by MongoDB Inc. You can use MongoDB to quickly develop applications by using the features that are provided, such as data scaling, auto-sharding, and high availability. You can also use MongoDB to store different types of document objects, such as JSON files, with a key-value pair.
Using MongoDB service

You can use the Node.js or Java sample applications to try the MongoDB service.

To use the MongoDB service in your application, complete the following steps:

    Manage MongoDB service instance by using the command line interface.
        Create a MongoDB service instance by typing the cf create-service command:

        $ cf create-service mongodb 100 mongodb01

        The cf create-service command creates a MongoDB service instance with a name of mongodb01 in your space. You can use the cf services command to list all available service instances that you created.
        Bind the service to your application by typing the cf bind-service command:

        $ cf bind-service AppName mongodb01

    Connect to the MongoDB service in your application.

    After you bind a MongoDB service instance to the application, the following configuration is added to your VCAP_SERVICES environment variable:

    { "mongodb-2.2": [ { "name": "mongodb-76b24", "credentials": { "hostname": "10.0.116.49", "host": "10.0.116.49", "port": 10001, "username": "be879069-b273-4656-b5fb-3daa5c508044", "password": "f268582e-0a52-42a8-9b97-66889a9cb662", "name": "76ea370c-8678-4c51-b3cf-a0cd722ed93a", "db": "db", "url": "mongodb://be879069-b273-4656-b5fb-3daa5c508044:f268582e-0a52-42a8-9b97-66889a9cb662@10.0.116.49:10001/db" } } ] }

    The configuration for the MongoDB service instance in the VCAP_SERVICES environment variable includes the following information:

    hostname
        The host name of the MongoDB server. 
    host
        The IP address of the MongoDB server.
    port
        The port number of the MongoDB server.
    name
        The user name for authentication.
    password
        The password for authentication.
    db
        The database name.
    url
        The connection URL. 

    To obtain the service credentials that are required to connect to the MongoDB instance, use the following code snippet:

    if (process.env.VCAP_SERVICES) { var env = JSON.parse(process.env.VCAP_SERVICES); var mongo = env['mongodb-2.2'][0].credentials; } else { var mongo = { "username" : "user1", "password" : "secret", "url" : "mongodb://user1:secret@localhost:27017/test" };

    The code above checks whether the VCAP_SERVICES environment variable exists. If the VCAP_SERVICES environment variable exists, the application uses the properties of the redis variable to access the database. If the VCAP_SERVICES environment variable does not exist, a local MongoDB service instance is used.
    Interact with the MongoDB service instance to put and get object.

    You can interact with the MongoDB service instance by using the service credentials; for example, you can read, write, or update the service instance. The following example shows how to write a string object to the MongoDB service instance:

    var create_message = function(req, res) { require('mongodb').connect(mongo.url, function(err, conn) { var collection = conn.collection('messages'); // create message record var parsedUrl = require('url').parse(req.url, true); var queryObject = parsedUrl.query; var name = (queryObject["name"] || 'Bluemix'); var message = { 'message': 'Hello, ' + name, 'ts': new Date() }; collection.insert(message, {safe:true}, function(err){ if (err) { console.log(err.stack); } res.writeHead(200, {'Content-Type': 'text/plain'}); res.write(JSON.stringify(message)); res.end('\n'); }); }); }

    After you write a string object to the MongoDB service instance, you can retrieve JSON objects from the MongoDB service instance by using the following code snippet:

    var list_message = function(req, res) { require('mongodb').connect(mongo.url, function(err, conn) { var collection = conn.collection('messages'); // list messages collection.find({}, {limit:10, sort:[['_id', 'desc']]}, function(err, cursor) { cursor.toArray(function(err, items) { res.writeHead(200, {'Content-Type': 'text/plain'}); for (i=0; i < items.length; i++) { res.write(JSON.stringify(items[i]) + "\n"); } res.end(); }); }); }); }

    Push your application to Bluemix™.

    After you finish coding, you can deploy your application to the Bluemix environment. To deploy an application, enter into the root directory of the application, and then type the following command:

    $ cf push

    Optional: Unbind or delete a MongoDB service instance.

    To unbind a service instance from an application, you can use the cf unbind-service command. To delete a service instance, use the cf delete-service command.

Binding a NoSQL (Mongo) database to Liberty applications

If you push an application or a packaged Liberty server and bind to a NoSQL database, the Liberty buildpack automatically generates or updates Mongo-related server.xml entries.
Pushing an Application

When you push an application, the Liberty buildpack generates a server.xml file. If you bind your application to a NoSQL database, the Liberty buildpack generates the associated server.xml stanzas that are required to access the database. Additionally, the Liberty buildpack provides the required JAR files of the client driver. In summary, the Liberty buildpack:

    Generates a <mongo> stanza.
    Generates a <mongoDB> stanza.
    Generates a <library> stanza with an embedded <fileset> stanza.
    Adds the mongodb-2.0 feature to the <featureManager> stanza.
    Adds the classloader for the Mongo library to the <application> stanza.

The Liberty buildpack must generate a jndiName in the <mongoDB> stanza. Because the Liberty buildpack does not have access to the JNDI name that is used by the application, it generates a JNDI name with a well-known convention of mongo/<service_name> where the <service_name> is the name of the bound service.

For example, if you bind a Mongo service that is named mymongo to the application, the Liberty buildpack generates a JNDI name of mongo/mymongo. When you develop the application and create the NoSQL database service, you must ensure that the JNDI name that is used by the application matches the JNDI name that is generated by the Liberty buildpack.

The following example shows the generated configuration when an application is pushed and bound to a Mongo service named mymongo.

<featureManager> <feature>webProfile-6.0</feature> <feature>jaxrs-1.1</feature> <feature>mongodb-2.0</feature> </featureManager> <application context-root='/' location='../../../../../' name='myapp' type='war'> <classloader commonLibraryRef='mongo-library'> </application> <mongo id='mongo-mymongo' libraryRef='mongo-library' password='${cloud.services.mymongo.connection.password}' user='${cloud.services.mymongo.connection.username}'> <hostNames>${cloud.services.mymongo.connection.host}</hostNames> <ports>${cloud.services.mymongo.connection.port}</ports> </mongo> <mongoDB databaseName='${cloud.services.mymongo.connection.db}' id='mongo-mymongo-db' jndiName='mongo/mymongo' mongoRef='mongo-mymongo'> <library id='mongo-library'> <fileset dir='${server.config.dir}/lib' id='mongo-fileset' includes='mongo-java-driver-2.11.3.jar'> </library>

Pushing a packaged server

When you push a packaged server, the Liberty buildpack detects and uses the server.xml file in the compressed file that is pushed. When you bind the packaged server to a NoSQL database, the Liberty buildpack checks the server.xml file for <mongo> and <mongoDB> entries that correspond to the bound database.

    If the Liberty buildpack does not find the <mongo> and <mongoDB> stanzas, they are created and added to the server.xml file.
    If the Liberty buildpack finds both the <mongo> and <mongoDB> stanzas, they are checked and updated as required to work in the cloud environment.
    If the Liberty buildpack finds one of the stanzas but not the other, the configuration is invalid.

If you provide configuration in your server.xml file, the configuration must be coherent. The <mongo>, <mongoDB>, <library>, and <fileset> stanzas must be configured properly. The <featureManager> must contain the mongodb-2.0 feature, and the <application> or <webApplication> stanza must contain the classloader that includes the library. Additionally, your configuration stanzas must contain configuration ids as described in the next section.

The Liberty buildpack searches for existing configuration by using the configuration IDs in the stanzas. The following algorithms are used to generate the configuration IDs that are used in the search:

    The <mongo> search uses a configuration ID of <service_type>-<service_name>.
    The <mongoDB> search uses a configuration ID of <service_type>-<service_name>.
    The <library> search uses a configuration ID of <service_type>-library.
    The <fileset> search uses a configuration ID of <service_type>-fileset.

The <service_type> is a lowercase string constant. For Mongo, the service_type is mongo. The <service_name> is the name of the service. For example, if you create a Mongo service with the name "mymongo", "mymongo" is the service_name and the mongo stanza ID is 'mongo-mymongo'. If you provide configuration stanzas, your configuration IDs must be created with the necessary algorithms. Failure to comply with these conventions results in errors.

If the Liberty buildpack finds existing configuration, it checks and updates if necessary. For example, the buildpack might update the following stanzas and attributes:

    The user and password attributes in the <mongo> stanza. Additionally, the nested <hostNames> and <ports> elements might be updated.
    The databaseName attribute of the <mongoDB> stanza.
    The dir and includes attributes in the <fileset> stanza.

Note: In this case, the Liberty buildpack does not update the jndiName attribute.

You can provide your own client driver JAR files. The client driver JAR files must be placed in the usr/servers/<servername>/lib directory. If you do not provide client driver JAR files, the Liberty buildpack provides the files. Any client JAR files that you provide must use the standard names that are established by the providing vendor. You cannot rename client JAR files.

The following code is a sample of Mongo configuration that might be provided in the local server.xml file. This sample assumes that when the packaged server is pushed to the cloud, it is bound to a Mongo database named "mymongo". In this case, the jndiName attribute can be whatever value you prefer. The value of the jndiName attribute is independent of the service name.

<mongo id="mongo-mymongo" libraryRef="mongo-library"> <hostNames>;localhost</hostNames> <ports>12707</ports> </mongo> <mongoDB id="mongo-mymongo-db" databaseName="test" jndiName="TestMongo" mongoRef="mongo-mymongo"> <library id="mongo-library"> <fileset id='mongo-fileset' dir='c:/mongoDB' includes='mongo-java-driver-2.11.3.jar'> </library>

When the server.xml file is pushed to the cloud, the Liberty buildpack updates the Mongo configuration stanzas as follows:

<mongo id='mongo-mymongo' libraryRef='mongo-library' password='${cloud.services.mymongo.connection.password}' user='${cloud.services.mymongo.connection.username}'> <hostNames>${cloud.services.mymongo.connection.host}</hostNames> <ports>${cloud.services.mymongo.connection.port}</ports> </mongo> <mongoDB databaseName='${cloud.services.mymongo.connection.db}' id='mongo-mymongo-db' jndiName='TestMongo' mongoRef='mongo-mymongo'/> <library id='mongo-library'> <fileset dir='${server.config.dir}/lib' id='mongo-fileset' includes='mongo-java-driver-2.11.3.jar'/> </library>

