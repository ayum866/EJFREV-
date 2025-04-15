# EJFREV-
DF G
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Multi-Page Messaging Platform</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        .message {
             border-radius: 0.5rem;
             padding: 0.75rem;
             margin-bottom: 0.5rem;
             background-color: #f0f0f0;
             color: #333;
             max-width: 80%;
             word-wrap: break-word;
        }
        .message.sent {
            background-color: #e0f7fa;
            color: #00838f;
            margin-left: auto;
            text-align: right;
        }
        .message.received {
            background-color: #f0f4c3;
            color: #6d4c41;
            margin-right: auto;
            text-align: left;
        }
    </style>
</head>
<body class="bg-gray-100 flex flex-col items-center justify-start min-h-screen py-10">
    <h1 class="text-3xl font-semibold text-blue-600 mb-8">Real-Time Messaging Platform (No DB)</h1>
    <div id="message-container" class="bg-white rounded-lg shadow-md p-6 w-full max-w-2xl mb-6 space-y-4 overflow-y-auto" style="height: 400px;">
        <p class="text-gray-700">Messages will appear here:</p>
    </div>
    <div id="typing-indicator" class="w-full max-w-2xl mb-2 text-gray-500 italic">
    </div>
    <form id="message-form" class="w-full max-w-2xl flex flex-col sm:flex-row gap-3 p-4">
        <input type="text" id="message-input" placeholder="Enter your message..." required
               class="flex-1 rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 p-4" />
        <button type="submit" class="bg-blue-500 hover:bg-blue-600 text-white rounded-md shadow-md py-4 px-6
                       transition duration-300 ease-in-out font-semibold">Send Message</button>
    </form>
    <div class="mt-4">
        <a href="users.html" class="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded">Go to Users Page</a>
    </div>
  <script src="https://cdn.socket.io/4.3.2/socket.io.min.js"></script>
    <script>
  const messageForm = document.getElementById('message-form');
        const messageInput = document.getElementById('message-input');
        const messageContainer = document.getElementById('message-container');
        const typingIndicator = document.getElementById('typing-indicator');
        const socket = io();
        const userId = `user-${Math.random().toString(36).substring(7)}`;
        let isTyping = false;
        let typingTimeout;

   function createMessageElement(text, type) {
            const messageElement = document.createElement('div');
            messageElement.classList.add('message', type);
            messageElement.textContent = text;
            return messageElement;
        }

   function addMessageToContainer(messageElement) {
            messageContainer.appendChild(messageElement);
            messageContainer.scrollTop = messageContainer.scrollHeight;
        }

   messageForm.addEventListener('submit', (event) => {
            event.preventDefault();
            const messageText = messageInput.value.trim();
            if (messageText === '') return;
            const messageElement = createMessageElement(messageText, 'sent');
            addMessageToContainer(messageElement);
            socket.emit('message', { senderId: userId, text: messageText });
            storeMessage(userId, messageText, 'sent');
            messageInput.value = '';
            isTyping = false;
            socket.emit('typing', { senderId: userId, isTyping: false });
            typingIndicator.textContent = '';
        });

 messageInput.addEventListener('input', () => {
            if (messageInput.value.trim() && !isTyping) {
                isTyping = true;
                socket.emit('typing', { senderId: userId, isTyping: true });
                typingIndicator.textContent = 'You are typing...';
                typingTimeout = setTimeout(() => {
                    isTyping = false;
                    socket.emit('typing', { senderId: userId, isTyping: false });
                    typingIndicator.textContent = '';
                }, 2000);
            } else if (!messageInput.value.trim() && isTyping) {
                isTyping = false;
                socket.emit('typing', { senderId: userId, isTyping: false });
                typingIndicator.textContent = '';
                clearTimeout(typingTimeout);
            }
        });
   socket.on('message', (data) => {
            if (data.senderId !== userId) {
                const messageElement = createMessageElement(data.text, 'received');
                addMessageToContainer(messageElement);
                storeMessage(data.senderId, data.text, 'received');
            }
        });
socket.on('typing', (data) => {
            if (data.senderId !== userId) {
                typingIndicator.textContent = data.isTyping ? `${data.senderId} is typing...` : '';
            }
        });

  function loadMessages() {
            const storedMessages = localStorage.getItem('messages');
            if (storedMessages) {
                try {
                    const messages = JSON.parse(storedMessages);
                    messageContainer.innerHTML = '';
                    messages.forEach(message => {
                        const messageElement = createMessageElement(message.text, message.sender === userId ? 'sent' : 'received');
                        addMessageToContainer(messageElement);
                    });
                } catch (error) {
                    console.error('Error parsing stored messages:', error);
                    localStorage.removeItem('messages');
                }
            }
        }

  function storeMessage(sender, text, type) {
            let messages = [];
            const storedMessages = localStorage.getItem('messages');
            if (storedMessages) {
                try{
                    messages = JSON.parse(storedMessages);
                } catch(e){
                    console.error("Error parsing stored messages", e);
                    localStorage.removeItem('messages');
                }
            }
            messages.push({ sender, text, type, timestamp: new Date().toISOString() });
            localStorage.setItem('messages', JSON.stringify(messages));
        }
        loadMessages();
    </script>
</body>
</html>
