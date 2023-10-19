# week-4-intermediate-javascript
React native
React Native is an open-source framework for building mobile applications developed by Facebook. It allows developers to build mobile applications using JavaScript and React. React Native enables developers to use React, a popular JavaScript library for building user interfaces, to create mobile applications that can run on both Android and iOS platforms.
Here are some key points about React Native:
Cross-Platform Development: One of the main advantages of React Native is its ability to create cross-platform mobile apps. This means developers can write code once and run it on both Android and iOS devices, saving time and effort compared to developing separate native apps for each platform.
Component-Based Architecture: React Native applications are built using components. Each component is a self-contained module that represents a part of the user interface. These components can be reused and combined to build complex user interfaces, making the development process more modular and efficient.
Native Performance: React Native does not use web views for rendering the user interface. Instead, it uses native components, allowing applications to achieve performance that is comparable to native apps. React Native bridges JavaScript code and native components, enabling smooth interactions and animations.
Hot Reloading: React Native includes a feature called Hot Reloading, which allows developers to see the changes they make in the code in real-time on the emulator or the physical device. This speeds up the development process by eliminating the need to recompile and reload the entire app after every change.
Large and Active Community: React Native has a large and active community of developers. There are extensive resources, libraries, and plugins available, making it easier for developers to find solutions to common problems and share knowledge.
Third-Party Libraries: React Native allows the use of third-party libraries and plugins, enabling developers to add a wide range of functionalities to their applications without having to build everything from scratch.

Installing Dependencies
NextJS — A framework for building server-side rendered(SSR) React applications with ease. It handles most of the challenges that come with building SSR React apps.
Pusher  — Pusher is a technology for building apps with varying real-time needs like push notifications and pub/sub messaging. It is the engine behind the real-time ability of our comments widget.
Sentiment — Sentiment is a module that uses the AFINN-165 wordlist and Emoji Sentiment Ranking to perform sentiment analysis on arbitrary blocks of the input text.
React — A very popular JavaScript DOM rendering framework for building scalable web applications using a component-based architecture.
Create a .env file in the root directory of your application and add your application details as follows:
PUSHER_APP_ID=YOUR_APP_ID
   PUSHER_APP_KEY=YOUR_APP_KEY
   PUSHER_APP_SECRET=YOUR_APP_SECRET
   PUSHER_APP_CLUSTER=YOUR_APP_CLUSTER
Create a Next.js configuration file named next.config.js in the root directory of your application with the following content:
/* next.config.js */
   const webpack = require('webpack');
   require('dotenv').config();
   module.exports = {
     webpack: config => {
      const env = Object.keys(process.env).reduce((acc, curr) => {
      acc[[Math Processing Error]] = JSON.stringify(process.env[curr]);
         return acc;
       }, {});
       config.plugins.push(new webpack.DefinePlugin(env));  
       return config;
     }
   };

Setting up the server
/* server.js */
   const cors = require('cors');
   const next = require('next');
   const Pusher = require('pusher');
   const express = require('express');
   const bodyParser = require('body-parser');
   const dotenv = require('dotenv').config();\
   const Sentiment = require('sentiment');
   const dev = process.env.NODE_ENV !== 'production';
   const port = process.env.PORT || 3000;
   const app = next({ dev });
   const handler = app.getRequestHandler();
   const sentiment = new Sentiment();
   // Ensure that your pusher credentials are properly set in the .env file
   // Using the specified variables
   const pusher = new Pusher({
     appId: process.env.PUSHER_APP_ID,
     key: process.env.PUSHER_APP_KEY,
     secret: process.env.PUSHER_APP_SECRET,
     cluster: process.env.PUSHER_APP_CLUSTER,
     encrypted: true
   });
   app.prepare()
     .then(() => {
       const server = express();
       server.use(cors());
       server.use(bodyParser.json());
       server.use(bodyParser.urlencoded({ extended: true }));
       server.get('*', (req, res) => {
         return handler(req, res);
       });
       server.listen(port, err => {
         if (err) throw err;
         console.log();
       });
     })
     .catch(ex => {
       console.error(ex.stack);
       process.exit(1);
     });

Pusher app
Pusher is a hosted API service which makes adding real-time data and functionality to web and mobile applications seamless. Pusher works as a real-time communication layer between the server and the client. It maintains persistent connections at the client using WebSockets, as and when new data is added to your server.

Building react native chat app
Setup React App:
If you haven't already, create a new React app using Create React App.  Create Components:
Inside the src folder, create two new components: Chat.js and Message.js. Style Your Components:
Create a CSS file (e.g., styles.css) in the src folder to style your components. Integrate Components:
Edit App.js to integrate the Chat component.Run Your App:
In the terminal, inside your project folder, run:

Building the chat component
Create a new Chat.js file inside the components directory and add the following content:
/* components/Chat.js */  
   import React, { Component, Fragment } from 'react';
   import axios from 'axios';
   import Pusher from 'pusher-js';  
   class Chat extends Component {  
     state = { chats: [] }   
     componentDidMount() {   
       this.pusher = new Pusher(process.env.PUSHER_APP_KEY, {
         cluster: process.env.PUSHER_APP_CLUSTER,
         encrypted: true
       });      
       this.channel = this.pusher.subscribe('chat-room');      
       this.channel.bind('new-message', ({ chat = null }) => {
         const { chats } = this.state;
         chat && chats.push(chat);
         this.setState({ chats });
       });      
       this.pusher.connection.bind('connected', () => {
         axios.post('/messages')
           .then(response => {
             const chats = response.data.messages;
             this.setState({ chats });
           });
       });   
     }    
     componentWillUnmount() {
       this.pusher.disconnect();
     }
   }
   export default Chat;

Adding the chat component to the index page
First, add the following line to the import statements in the pages/index.js file.
/* pages/index.js */
   // other import statements here ...
   import Chat from '../components/Chat';
Next, locate the render() method of the IndexPage component. Render the Chat component in the empty <section>element. It should look like the following snippet:
/* pages/index.js */
   <section className="col-md-4 position-relative d-flex flex-wrap h-100 align-items-start align-content-between bg-white px-0">
     { user && <Chat activeUser={user} /> }
   </section>

Adding the chat component to the index page
Modify the server.js file and add the following just before the call to server.listen()inside the then() callback function:

/* server.js */

   // server.get('*') is here ...  

   const chatHistory = { messages: [] };  

   server.post('/message', (req, res, next) => {

     const { user = null, message = '', timestamp = +new Date } = req.body;

     const sentimentScore = sentiment.analyze(message).score;    

     const chat = { user, message, timestamp, sentiment: sentimentScore };    

     chatHistory.messages.push(chat);

     pusher.trigger('chat-room', 'new-message', { chat });

   });

   server.post('/messages', (req, res, next) => {

     res.json({ ...chatHistory, status: 'success' });

   });  

   // server.listen() is here …

First, we created a kind of in-memory store for our chat history, to store chat messages in an array. This is useful for new clients that join the chat room to see previous messages. Whenever the Pusher client makes a POST request to the /messages endpoint on connection, it gets all the messages in the chat history in the returned response.

Displaying the chat message and deploying your chat app
Create a new ChatMessage.js file inside the components directory and add the following content to it.
/* components/ChatMessage.js */  
   import React, { Component } from 'react';  
   class ChatMessage extends Component {  
     render() {
       const { position = 'left', message } = this.props;
       const isRight = position.toLowerCase() === 'right';      
       const align = isRight ? 'text-right' : 'text-left';
       const justify = isRight ? 'justify-content-end' : 'justify-content-start';      
       const messageBoxStyles = {
         maxWidth: '70%',
         flexGrow: 0
       };      
       const messageStyles = {
         fontWeight: 500,
         lineHeight: 1.4,
         whiteSpace: 'pre-wrap'
       };      
       return <div className={}>
         <div className="bg-light rounded border border-gray p-2" style={messageBoxStyles}>
           <span className={} style={messageStyles}>
             {message}
           </span>
         </div>
       </div>
     }
   }
   export default ChatMessage;
Finally, we will modify the components/Chat.js file to render the chat messages from the state. Make the following changes to the Chatcomponent. First add the following constants before the class definition of the Chatcomponent. Each constant is an array of the code points required for a particular sentiment emoji. Also ensure to import the ChatMessagecomponent.
/* components/Chat.js */
   // Module imports here ...
   import ChatMessage from './ChatMessage';
   const SAD_EMOJI = [55357, 56864];
   const HAPPY_EMOJI = [55357, 56832];
   const NEUTRAL_EMOJI = [55357, 56848];
     // Chat component class here …

Then, add the following snippet between the chat header <div> and the chat message box <div> we created earlier in the Chat component.
/* components/Chat.js */  
   {/** CHAT HEADER HERE **/}  
   <div className="px-4 pb-4 w-100 d-flex flex-row flex-wrap align-items-start align-content-start position-relative" style={{ height: 'calc(100% - 180px)', overflowY: 'scroll' }}>  
     {this.state.chats.map((chat, index) => {    
       const previous = Math.max(0, index - 1);
       const previousChat = this.state.chats[previous];
       const position = chat.user === this.props.activeUser ? "right" : "left";      
       const isFirst = previous === index;
       const inSequence = chat.user === previousChat.user;
       const hasDelay = Math.ceil((chat.timestamp - previousChat.timestamp) / (1000 * 60)) > 1;      
       const mood = chat.sentiment > 0 ? HAPPY_EMOJI : (chat.sentiment === 0 ? NEUTRAL_EMOJI : SAD_EMOJI);      
       return (
         <Fragment key={index}>        
           { (isFirst || !inSequence || hasDelay) && (
             <div className={} style={{ fontSize: '0.9rem' }}>
               <span className="d-block" style={{ fontSize: '1.6rem' }}>
                 {String.fromCodePoint(...mood)}
               </span>
               <span>{chat.user || 'Anonymous'}</span>
             </div>
           ) }          
           <ChatMessage message={chat.message} position={position} />          
         </Fragment>
       );      
     })} 
   </div>  
   {/** CHAT MESSAGE BOX HERE **/}
