## node-red-contrib-postgres-multi

A [Node-RED](http://nodered.org) node to query [PostgreSQL](http://www.postgresql.org/), with multiple query support.

Based on [node-red-contrib-postgres](https://github.com/krisdaniels/node-red-contrib/tree/master/node-red-contrib-postgres) by Kris Daniels.

### Requirements

This module uses JavaScript features only found in Node versions 8+.

### Install

Run the following command in the root directory of your Node-RED install

    npm install node-red-contrib-postgres-multi

### Usage

Assemble your queries as an array of objects on msg.payload:

    msg.payload = [
        {
            query: 'begin',
        },
        {
            query: 'delete from mytable',
        },
        {
            query: 'insert into mytable (id, message) values (1, $hello), (2, $world)',
            params: {
                hello: 'Hi there',
                world: 'O\'Rorke',
            },
        },
        {
            query: 'commit',
        },
    ];

As you can see, this structure allows you to create your own transaction boundaries.

If you want the output of one or more queries, check the 'Receive output' box in the postgres node
and include an `output: true` member in the query object(s) you expect results from.
The results are then set on the msg.payload property of the outbound message.

    msg.payload = [
        {
            query: 'select message from mytable where id=$1',
            params: [1],
            output: true,
        },
        {
            query: 'select message from mytable where id=$1',
            params: [2],
            output: true,
        },
    ];

    # output:

    [
        ['Hi there'],
        ['O\'Rorke']
    ]

### Example DB

    create table mytable
    (
        id int not null,
        message char(20)
    );

### Example node-red flow

Import the flow below in an empty sheet in nodered

    [{"id":"168f2030.e970e","type":"postgresdb","z":"d1b19967.2e4e68","hostname":"localhost","port":"5432","db":"postgres"},{"id":"37e19274.c81e6e","type":"inject","z":"d1b19967.2e4e68","name":"trigger","topic":"","payload":"","payloadType":"date","repeat":"","crontab":"00 12 * * *","once":false,"x":135,"y":78,"wires":[["ec61e7da.139e18"]]},{"id":"3bffa50b.c4005a","type":"template","z":"d1b19967.2e4e68","name":"format query","field":"payload","fieldType":"msg","format":"handlebars","syntax":"mustache","template":"select * from table1 where field2 > $param1","x":528,"y":76,"wires":[["e3ff7e9.f1c008"]]},{"id":"e3ff7e9.f1c008","type":"postgres","z":"d1b19967.2e4e68","postgresdb":"168f2030.e970e","name":"","output":true,"outputs":1,"x":695,"y":72,"wires":[["9e662745.6199d8","208d526c.df72ae"]]},{"id":"9e662745.6199d8","type":"debug","z":"d1b19967.2e4e68","name":"query output","active":false,"console":"false","complete":"true","x":877,"y":198,"wires":[]},{"id":"208d526c.df72ae","type":"http request","z":"d1b19967.2e4e68","name":"","method":"POST","ret":"txt","url":"http://localhost:1880/incoming_data","x":1009,"y":79,"wires":[["e48e88e6.1b7178"]]},{"id":"e48e88e6.1b7178","type":"debug","z":"d1b19967.2e4e68","name":"http result","active":false,"console":"false","complete":"payload","x":1192,"y":79,"wires":[]},{"id":"ec61e7da.139e18","type":"function","z":"d1b19967.2e4e68","name":"setup params","func":"msg.queryParameters = msg.queryParameters || {};\nmsg.queryParameters.param1 = 1;\nreturn msg;","outputs":1,"noerr":0,"x":328,"y":74,"wires":[["3bffa50b.c4005a"]]},{"id":"f7a7d4a7.085828","type":"http in","z":"d1b19967.2e4e68","name":"","url":"/incoming_data","method":"post","swaggerDoc":"","x":231,"y":494,"wires":[["7b5aa85e.84a558","ac9432ad.536bd"]]},{"id":"687b82be.97847c","type":"http response","z":"d1b19967.2e4e68","name":"","x":835,"y":510,"wires":[]},{"id":"7b5aa85e.84a558","type":"debug","z":"d1b19967.2e4e68","name":"http input","active":false,"console":"false","complete":"payload","x":450,"y":602,"wires":[]},{"id":"ac9432ad.536bd","type":"function","z":"d1b19967.2e4e68","name":"reply to http call","func":"msg.payload='reply from http';\nreturn msg;","outputs":1,"noerr":0,"x":599,"y":444,"wires":[["687b82be.97847c"]]}]