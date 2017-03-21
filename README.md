
# Interconnect - Build a cognitive app in 20 minutes

## Introduction

In this short workshop we will experience how easy it is to build a (simple) cognitive application. After finishing this lab you can extend and enhance this application in any way you like. 

This Application makes use of [Node-RED](http://nodered.org), which is a visual tool to wire things together, with just limited coding. 

We use the NODE-RED boilerplate in [Bluemix](http://Bluemix.net) which consists of the Node-RED application (node.js) and a Cloudant database. 

We will build an cognitive app, with the [Watson tone analyzer](https://www.ibm.com/watson/developercloud/tone-analyzer.html) service 

With Watson Tone Analyzer you can analyze the tone of voice of written text. The source of our text will be Twitter.  

At last we will display the output on a dashboard. 

As already mentioned this is just a starter application to show you how easy it is to use Node-RED and the Watson API’s. When you have this application running you can extend the application with more functionality if needed. 

In step six you will find the complete flow of this lab.

Please make sure you stop the app after this lab., tone analyzer will continue working and will take a lot of your resources especially when you have a paid account!

## Step one: Deploy Node-RED on Bluemix

You need a Bluemix account, if you don't have one yet, follow the steps to can create an account [here](http://Bluemix.net). 

When you are logged in to Bluemix:

1. Click to 'Catalog' 
1. Click on “Apps”
1. Click on “Boilerplates”
1. Click on the 'Node-RED starter'. 
 On the next screen you must give the application unique name, which wil be part of the URL to the application. 
1. Then click 'Create'. 

Right now the Node-RED application is being created and deployed. This will take a few minutes. The application consists of a Node-JS runtime and a Cloudant NoSQL database.

## Step two: Add Watson Tone Analyzer service

When the application is started, click on 'Connections' in the left menu

2. Click 'Connect new'
2. Click on 'Watson' (on the left)
2. Click Tone Analyzer'
2. Click 'Create'
 Now the tone analyser service is being added to the application, so that we can use it in Node-RED.
2. Click 'Restage'
 The application will be restarted

When finished click on the URL which opens up your Node-RED application. Then click on 'Go to your Node-RED flow editor'

## Step three: Build your application

Now it is time to build your application by dragging and dropping nodes to the canvas.
On the left side of the screen you see the nodes.
Every node has it’s own functionality and parameters.

3. We start with a **twitter in** node, drag and drop it to the canvas.
3. Double click the twitter in node you just added, fill in your twitter handle (by clicking the pencil icon) and a keyword. This keyword will be searched on. (this node is only reading, not posting anything)
3. Then add the **Tone Analyzer V3** node and wire it to twitter node
 No configuration needed as this time
3. Add 2 Function nodes
3. Wire the first to the Tone Analyzer node and wire the second tot the first Function node
 We use a simple function because Tone AnalyZer will give five different tones and we will use only one. This first function node will extract the  tone with the highest score. 
3. Give the first node a name( by double clicking on the node), for instance: 'Find top tone score' and add the following code to the first node:

```
if(msg.response.document_tone) {
 
 // Get the Social Summary tones.
 var tones = msg.response.document_tone.tone_categories[0].tones;
 var score = 0;
 
 // Find the highest tone.
 for(var i in tones) {
 if(tones[i].score > score) {
 msg.toptone = tones[i].tone_id; 
 score = tones[i].score;
 }
 }
 
 // Update the scoreboard, adding one to the category with the highest tone.
 // context.global.scores[msg.toptone]++;
 
 return msg;
}
```
Tone analyzer will give results for more then one category. We will only use the top tone score. The code above will derive the top tone score. the code below will send the top tone score to the payload.

3. give the second node a name, for instance: 'Combine tweet and tone' and add the following code:
```
msg.payload = {
 tweet: msg.tweet, 
 tone: msg.response,
 toptone: msg.toptone
};

return msg;
```

You just build your first application!.

To save and deploy the flow you just build to Bluemix, click on the 'Deploy' button. 

Next step is to display the results of the twitter analysis. 

## Step four: Add dashboard nodes 

First check if if the dashboard nodes are available. To do so, scroll down in the nodes palette, on the left side of the screen. If there is a section called **Dashboard** you can continue to the step five. Otherwise follow the follwoing steps to install the dashboard nodes.

4. Go to the top right ‘hamburger menu’ then click on ‘Manage palette’, then
4. Click on the 'Install-tab'
4. Search for 'node-red-dashboard'
4. When found, click 'Install' and on 'Install' on the pop-up screen.
4. Then click 'Done'

When the installation finished you should see the nodes in the nodes palette

## Step five: Build the dashboard

on the dashboard we will show the tweet and the highest tone score

5. Got to the Dashboad section of the palette and add two Text nodes 
5. Wire both to the second Function node you created earlier
5. Open the first Text node, by double clicking.
5. Create a group where both dashboard nodes will be part of, by clicking on the pencil icon. 
5. Then create a new 'ui-tab' by clicking on the pencil icon. in the next screen leave everything default and click 'Add' 
5. You can name this node “tweet” because this node will display the tweet on the dashboard. You can fill this in the name in the "label' field
5. You need also to add the value format, this is the extacted text of the tweet
 The value format is : ```{{msg.payload.tweet.text}}```

5. The second node will contain the tone score of the tweet. As a group, use the name of the group you created earlier, label can be 'Tone score' and the value format: {{msg.toptone}}

5. Then click 'Deploy'

To see the results use your URL with ```/ui``` for instance: https://Watsonlab.mybluemix.net/ui 
When a new tweet is analyzed, you wil see it appear on the Dashboard, with the corresponding Tone Score.

Maybe the lay-out can be better, but you can change that later.

Then you are finished and have your cognitive application running including the dashboard! 


## Step 6 (optional): Extend your application

If you got time you can extend the application. In the sample flow below, I have added a filter, to only analyze tweets in English, and a delay node to analyze less messages per minute.

To get a better layout, you can change parameters of the dashboard nodes or on the right side of the screen, there will be an extra tab called dashboard.

For debugging you can add debug nodes to every node to see what is happening.

It is also possible to import the complete flow.

6. Copy this flow:
```
[{"id":"484d9b3e.99761c","type":"delay","z":"cc74818a.3d2ff","name":"","pauseType":"queue","timeout":"10","timeoutUnits":"seconds","rate":"1","nbRateUnits":"5","rateUnits":"minute","randomFirst":"1","randomLast":"5","randomUnits":"seconds","drop":false,"x":389.5,"y":121.75,"wires":[["b63393c3.49b34"]]},{"id":"3f639787.ee73b8","type":"switch","z":"cc74818a.3d2ff","name":"","property":"lang","propertyType":"msg","rules":[{"t":"eq","v":"en","vt":"str"},{"t":"neq","v":"en","vt":"str"}],"checkall":"true","outputs":2,"x":189.5,"y":124.75,"wires":[["ab999718.7b1188","484d9b3e.99761c"],["2bd04bc1.dab424"]]},{"id":"b63393c3.49b34","type":"watson-tone-analyzer-v3","z":"cc74818a.3d2ff","name":"","tones":"emotion","sentences":"true","contentType":"false","x":590,"y":122.25,"wires":[["8ce05eb8.9f52e8","f6de2113.911f28"]]},{"id":"dbdcbc8d.a27958","type":"twitter in","z":"cc74818a.3d2ff","twitter":"","tags":"weather","user":"false","name":"","topic":"tweets","x":60,"y":124.5,"wires":[["71d5b271.714c24","4fc0ae96.12d92","3f639787.ee73b8"]]},{"id":"ab999718.7b1188","type":"debug","z":"cc74818a.3d2ff","name":"","active":true,"console":"false","complete":"false","x":364.5,"y":40.5,"wires":[]},{"id":"2bd04bc1.dab424","type":"debug","z":"cc74818a.3d2ff","name":"","active":true,"console":"false","complete":"false","x":334.25,"y":197.24996948242188,"wires":[]},{"id":"8ce05eb8.9f52e8","type":"function","z":"cc74818a.3d2ff","name":"Find Top Tone Score","func":"if(msg.response.document_tone) {\n \n // Get the Social Summary tones.\n var tones = msg.response.document_tone.tone_categories[0].tones;\n var score = 0;\n \n // Find the highest tone.\n for(var i in tones) {\n if(tones[i].score > score) {\n msg.toptone = tones[i].tone_id; \n score = tones[i].score;\n }\n }\n \n // Update the scoreboard, adding one to the category with the highest tone.\n // context.global.scores[msg.toptone]++;\n \n return msg;\n}","outputs":1,"noerr":0,"x":820.5,"y":118,"wires":[["dc7bd59.3902028","2b4de80d.3613b8"]]},{"id":"f6de2113.911f28","type":"debug","z":"cc74818a.3d2ff","name":"Toggle to show tone category scores","active":false,"console":"false","complete":"response.document_tone.tone_categories","x":821.9000244140625,"y":33.9666748046875,"wires":[]},{"id":"71d5b271.714c24","type":"debug","z":"cc74818a.3d2ff","name":"","active":false,"console":"false","complete":"lang","x":140.5,"y":55.5,"wires":[]},{"id":"4fc0ae96.12d92","type":"debug","z":"cc74818a.3d2ff","name":"Toggle to show tweets","active":false,"console":"false","complete":"tweet.text","x":217.24996948242188,"y":253.25,"wires":[]},{"id":"dc7bd59.3902028","type":"debug","z":"cc74818a.3d2ff","name":"Toggle to show top tone category","active":true,"console":"false","complete":"toptone","x":1015.0000610351562,"y":75.5,"wires":[]},{"id":"2b4de80d.3613b8","type":"function","z":"cc74818a.3d2ff","name":"Combine tweet/tone","func":"msg.payload = {\n tweet: msg.tweet, \n tone: msg.response,\n toptone: msg.toptone\n};\n\nreturn msg;","outputs":1,"noerr":0,"x":1069,"y":117,"wires":[["ab2386ab.28dc9","83fb32d7.e78168","7e2cf205.4e48b4","679995a9.1381d4","41ab3a0d.de352c"]]},{"id":"ab2386ab.28dc9","type":"debug","z":"cc74818a.3d2ff","name":"","active":false,"console":"false","complete":"payload.tweet.text","x":1333.5000610351562,"y":217.5,"wires":[]},{"id":"83fb32d7.e78168","type":"debug","z":"cc74818a.3d2ff","name":"","active":false,"console":"false","complete":"response.document_tone.tone_categories[0].tones","x":1218,"y":293.25,"wires":[]},{"id":"7e2cf205.4e48b4","type":"debug","z":"cc74818a.3d2ff","name":"","active":false,"console":"false","complete":"toptone","x":1351.5,"y":32.250030517578125,"wires":[]},{"id":"679995a9.1381d4","type":"ui_text","z":"cc74818a.3d2ff","group":"b02f6a63.c72918","order":2,"width":0,"height":0,"name":"","label":"tone score:","format":"{{msg.toptone}}","layout":"col-center","x":1309.5,"y":149.5,"wires":[]},{"id":"41ab3a0d.de352c","type":"ui_text","z":"cc74818a.3d2ff","group":"b02f6a63.c72918","order":1,"width":0,"height":0,"name":"","label":"tweet:","format":"{{msg.payload.tweet.text}}","layout":"col-center","x":1288,"y":84,"wires":[]},{"id":"b02f6a63.c72918","type":"ui_group","z":"","name":"Interconnect","tab":"72c3151c.92df5c","disp":true,"width":"6"},{"id":"72c3151c.92df5c","type":"ui_tab","z":"","name":"Home","icon":"dashboard"}]
```
6. Then go to the top right hamburger menu, select import , clipboard
6. Paste the text and then Import

Then you got the complete flow, the only thing you have to change is the twitter handle in the Twitter in node.

##Wrap up##

In this 20 minute workshop you got hopefully an idea on how to write a cognitive app with just little effort. This is surely not a complete application, but you can use this as a starting point to build a more advanced application, maybe in combination with other data sources and Watson services.

Please make sure you stop the app, tone analyzer will continue working and will take a lot of your resources especially when you have a paid account!
