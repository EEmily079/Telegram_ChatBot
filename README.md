Chat Bot Telegram >>>>>>>>>>>>>>>>>>>>>>>>>>>>

1. create chat bot in telegram 
Find botfatehr in Telegram then start 
create newbot by clicking /newbot > then give a name to that bot
identigy the bot name (must end the name with bot example : emily_bot)
then Telegram will reply with token. click on the link to access 
in this case is TELE-BOT-GOOLE

2. create folder and use VS code
in terminal connecting bot to local server 
npm init >> then keep enter for all the command and say yes to create 
install 2 packages >>> npm install axios express

 
3. create index.js file
	//import express
	const express = require ('express')
	//find the prot 
	const PORT = process.env.PORT || 4040;
	
	//start the server 
	const app = express();
	//response to all post with get request 
	app.use(express.json());
	app.post("*",async (req, res)=>{
    	res.send("Hello post");
	});
	app.get("*", async (req,res)=>{
    	res.send("Hello get");
	});
	
	//then listen those sending from the port 
	app.listen(PORT, function (err){
    	if(err) console.log(err);
    	console.log("Server listening to PORT", PORT);
	});
	
In package json comment the script of package json to start the server 
	"scripts": {
   		 "test": "echo \"Error: no test specified\" && exit 1",
   		 "start": "node index"
 		 },

then from terminal > start the sever > npm run start 

4.Since Telegram only will interact with https protocol API, and node only runs on local server 
to bridge the gap we use ngrok (Telegram > Ngrok > Node) 

confing ngrok with autherization token from comment promp 
go to nogrok file then paste it >> 
	ngrok config add-authtoken 2kdBU1Ga6MgbFABkTx6Kd17l6d3_3CV95peqRb1L4azNUUt9z

if we saw ngrok.yml file then this is good to go.
to connect with ngrok comment this >> ngrok http 4040 (prot number) 
then copy https// port nnumber to connect with telegram 


5. connect ngrok with telegram (core.telegram.org)
in telegram documentation > provide method for API webhook
https://api.telegram.org/bot${MY_TOKEN}/${method}

to update webhook go to postmen and set token (from telegram) and url(ngrok)
 https://api.telegram.org/bot7308093712:AAG_QCK_lv4e3i0GeYjbgeifhFQRNYTDUDc/setWebhook?
	url=https://e7f2-220-255-78-152.ngrok-free.app

6. to check what we are receiving add log for request body.
	app.post("*",async (req, res)=>{
    	console.log(req.body);
    	res.send("Hello post");
	});
then restart the server npm run start
send message from telegram and will show it in message log for request body.

Note : ngrok service always provide new URL when restart it ( so need to update in post man and test if restart ngrok)

7. Response back Message 
enable proper communication with the bot > create contoller folder > inside it create lib to handle all the request
	> inside lib create axios.js to be responsible for all the API calls.  
		//improt axios libry and bot token ( this is basic requirement for setting up the telegram bot)
		const axios = require ('axios');
		const MY_TOKEN = "7308093712:AAG_QCK_lv4e3i0GeYjbgeifhFQRNYTDUDc"
		const BASE_URL = `https://api.telegram.org/bot${MY_TOKEN}`;
		
		//create axios instance and export it. inside instance create 2 methods 
		//get () to make all the get calls and post() to make any http requests  
		function getAxiosInstance(){
    			return {
       			 get(method , params){
           		 return axios.get(`/${method}`,{
                	baseURL : BASE_URL,
                	params,
            		});
        	},

        	post(method, data){
            	return axios ({
                method : "post",
                baseURL : BASE_URL,
                url : `/${method}`,
                data,
           	 });

       		},
   	  };
	}
	module.exports = {axiosInstance : getAxiosInstance()};

inside lib create telegram.js file to communicate the telegram bot 
	//import axios to achieve sending message from telegram bot
	const { axiosInstance } = require("./axios");

	//this function to make a get calls using azios and sending back 
	//availabe message in the telegram API, messageText is JSON(chat_id and text)
	function sendMessage(messageObj, messageText){
    		return axiosInstance.get("sendMessage", {
        	chat_id: messageObj.chat.id,
        	text: messageText,
    		});
	}

	//takes message object and extrat the message text 
	
	function handleMessage(messageObj){
    		const messageText = messageObj.text || "";

	//if message strat with / it will recognise as command
	//then remove the / and record the rest of the string as command
    		if(messageText.charAt(0) === "/"){
        	const command = messageText.substr(1);

        	switch (command){
	//if command is strat we will send welcome message to user 
           	 case "start":
                return sendMessage(
                    messageObj,
                    "Hi! I am a bot. I can help you to get started"
                );
        //if command is unknown we simply return as don't know
                default:
                    return sendMessage(
                        messageObj , 
                        "Hey, I don't know that command"
                    )
       	 }
   	 }
    	  else {
        //we send some message to user 
        return sendMessage(messageObj, messageText);
   	 }
	}

	module.exports = {handleMessage};

8. in controller > create index.js file to imoport handle message 

	//to take all the input request we are getting to this server and handle them appropiately
	const {handleMessage} = require ("./lib/telegram");

	async function handler(req, method) {
    		//take the body from request 
    		const {body} = req;
    		if(body){

		//send that body message in handle message and return it
       		 const messageObj = body.message;
        	await handleMessage(messageObj);
    		}
		return;
	}

	module.exports = {handler};

this handle message method always return in axios instance which is in async operation. 
await resolution of send message method before returning it. 

9. Finally hook the handler method to the request in server starting file. 

index.js (outside node module) > instead of responding "Hello post" 
	pass request to handler to takecare of all the necessary actions.

		app.post("*",async (req, res)=>{
    		console.log(req.body);
    		res.send(await handler(req));
		});

		app.get("*", async (req,res)=>{
    		res.send(await handler(req));
		});

then restarting the bot > npm run start

if didn't received any response from the log config to ngrok ( restart all )

10.own command in botfather > /setcommends and choose our bot to set 
	then follow the instruction to create		
	command1 - Description
	command2 - Another description
 



