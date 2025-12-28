# Spur Chat Frontend

A clean and responsive Svelte-based chat UI for an AI-powered customer support system.
This frontend connects to the Spur Chat Backend and provides a real-time, session-based chat experience similar to modern support widgets.

Built with simplicity, clarity, and reliability in mind.

## Features

- Real-time chat interface
- Session-based conversation persistence
- Automatic chat history loading
- Typing indicator for AI responses
- Clear chat functionality
- Smooth auto-scroll behavior
- Mobile-friendly and responsive UI
- Minimal, classic support-chat design

## Tech Stack

- Svelte
- TypeScript
- Fetch API


## Project Overview

### The frontend:

- Stores sessionId in localStorage
- Restores chat history on page reload
- Communicates with backend REST APIs
- Displays user and AI messages with avatars
- Handles loading, error, and empty states gracefully

### Clone the Repository
```
git clone https://github.com/Harshpatil0508/spur-frontend.git
cd spur-frontend
```
### Environment Configuration

- Create a .env file in the project root:
```
PUBLIC_API_BASE_URL=https://your-backend-url
```

- Example:
```
PUBLIC_API_BASE_URL=http://localhost:3001
```

## Installation

### Install dependencies:
```
npm install
```

## Running the App
### Development
```
npm run dev
```

## The app will be available at:
```
http://localhost:5173
```

## Production Build
```
npm run build
```


## Chat Flow

- User types a message and presses Enter or clicks Send
- Message is sent to backend /chat/message
- Backend returns AI reply and sessionId
- sessionId is stored in localStorage
- On reload, chat history is fetched from /chat/history/:sessionId

## API Integration
### Send Message

- POST /chat/message


- Request body:
```
{
  "message": "Hello",
  "sessionId": "session-id"
}
```

### Fetch Chat History

- GET /chat/history/:sessionId

### UI Behavior
- User messages appear on the right
- AI messages appear on the left
- Avatars indicate sender
- Typing indicator shows while waiting for AI response
- Timestamps displayed per message
- Clear button resets session and chat state