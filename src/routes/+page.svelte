<script lang="ts">
  import { onMount } from 'svelte';

  type Message = {
    sender: 'user' | 'ai';
    text: string;
    createdAt?: string;
  };

  let messages: Message[] = [];
  let input = '';
  let loading = false;
  let sessionId: string | null = null;

  // Load chat history safely
  onMount(async () => {

    console.log("API BASE URL:", import.meta.env.PUBLIC_API_BASE_URL);

    sessionId = localStorage.getItem('sessionId');
    if (!sessionId) return;

    try {
      const res = await fetch(
        `${import.meta.env.PUBLIC_API_BASE_URL}/chat/history/${sessionId}`
      );

      if (!res.ok) {
        localStorage.removeItem('sessionId');
        sessionId = null;
        return;
      }

      const data = await res.json();
      messages = data.messages || [];
      scrollToBottom();
    } catch {
      console.error('Failed to load chat history');
    }
  });

  async function sendMessage() {
    if (!input.trim() || loading) return;

    const userText = input.trim();

    // Optimistic UI update
    messages = [
      ...messages,
      { sender: 'user', text: userText, createdAt: new Date().toISOString() }
    ];

    input = '';
    loading = true;

    try {
      // Only send sessionId if valid
      const payload: any = { message: userText };
      if (sessionId) payload.sessionId = sessionId;

      const res = await fetch(
        `${import.meta.env.PUBLIC_API_BASE_URL}/chat/message`,
        {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        }
      );

      const data = await res.json();

      if (data.sessionId) {
        sessionId = data.sessionId;
        localStorage.setItem('sessionId', sessionId);
      }

      messages = [
        ...messages,
        {
          sender: 'ai',
          text: data.reply,
          createdAt: new Date().toISOString()
        }
      ];
    } catch {
      messages = [
        ...messages,
        { sender: 'ai', text: 'Something went wrong.' }
      ];
    } finally {
      loading = false;
      scrollToBottom();
    }
  }

  function clearChat() {
    messages = [];
    sessionId = null;
    localStorage.removeItem('sessionId');
  }

  function scrollToBottom() {
    setTimeout(() => {
      const el = document.getElementById('chat');
      el?.scrollTo({ top: el.scrollHeight, behavior: 'smooth' });
    }, 50);
  }
</script>

<div class="chat">
  <div class="header">
    <h3>Support Chat</h3>
    <button class="clear" on:click={clearChat}>Clear</button>
  </div>

  <div id="chat" class="messages">
    {#each messages as msg}
      <div class="msg {msg.sender}">
        <div>{msg.text}</div>
        <small class="time">
          {msg.createdAt ? new Date(msg.createdAt).toLocaleTimeString() : ''}
        </small>
      </div>
    {/each}

    {#if loading}
      <div class="msg ai typing">Agent is typing…</div>
    {/if}
  </div>

  <div class="input-container">
    <input
      bind:value={input}
      placeholder="Ask a question…"
      on:keydown={(e) => e.key === 'Enter' && sendMessage()}
    />
    <button on:click={sendMessage} disabled={loading}>Send</button>
  </div>
</div>

<style>
  .header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0.75rem 1rem;
    background: #4a90e2;
    color: white;
  }

  .clear {
    background: transparent;
    border: 1px solid white;
    color: white;
    padding: 4px 10px;
    border-radius: 12px;
    cursor: pointer;
  }

  .clear:hover {
    background: rgba(255, 255, 255, 0.15);
  }

  .chat {
    max-width: 600px;
    margin: 2rem auto;
    height: 80vh;
    display: flex;
    flex-direction: column;
    border-radius: 12px;
    overflow: hidden;
    box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: #f9f9f9;
  }

  .messages {
    flex: 1;
    padding: 1rem;
    overflow-y: auto;
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }

  .msg {
    padding: 0.75rem 1rem;
    border-radius: 18px;
    max-width: 70%;
    word-wrap: break-word;
    line-height: 1.4;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  }

  .user {
    background: #dcf8c6;
    align-self: flex-end;
    border-bottom-right-radius: 4px;
  }

  .ai {
    background: #fff;
    align-self: flex-start;
    border-bottom-left-radius: 4px;
  }

  .input-container {
    display: flex;
    padding: 0.75rem 1rem;
    background: #eee;
    gap: 0.5rem;
  }

  input {
    flex: 1;
    padding: 0.75rem 1rem;
    border-radius: 20px;
    border: 1px solid #ccc;
    outline: none;
    font-size: 1rem;
  }

  button {
    padding: 0 1.5rem;
    border-radius: 20px;
    background: #4a90e2;
    color: white;
    font-weight: bold;
    cursor: pointer;
  }

  .typing {
    font-style: italic;
    color: #555;
  }

  .time {
    font-size: 0.7rem;
    color: #777;
    margin-top: 4px;
    text-align: right;
  }
</style>
