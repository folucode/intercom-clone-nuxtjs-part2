<template>
  <div class="center">
    <div class="contacts">
      <h2 style="text-decoration: underline; margin-bottom: 7rem">
        Chats
      </h2>
      <div class="contact">
        <li v-for="session in chatSessions" @click="getChat(session.sessionID)">
          <div class="name">
            Session ID
          </div>
          <div class="message">
            {{ session.sessionID }}
          </div>
        </li>
      </div>
    </div>
    <div class="chat">
      <div class='contact bar'>
        <div class='pic helpdesk'></div>
        <div class='name'>Help Desk</div>
      </div>
      <div class="messages" id="chat">
        <li v-for="message in messages">
          <b>{{ message.sender.toUpperCase() }}:</b>
          <div class='message'>{{ message.message }}</div>
        </li>
      </div>
      <div class="input">
        <input placeholder='Type your message here!' v-model="text" />
        <svg xmlns='http://www.w3.org/2000/svg' width='26' height='26' fill='currentColor' class='bi bi-send'
          viewBox='0 0 16 16' @click="sendMessage()">
          <path
            d='M15.854.146a.5.5 0 0 1 .11.54l-5.819 14.547a.75.75 0 0 1-1.329.124l-3.178-4.995L.643 7.184a.75.75 0 0 1 .124-1.33L15.314.037a.5.5 0 0 1 .54.11ZM6.636 10.07l2.761 4.338L14.13 2.576 6.636 10.07Zm6.787-8.201L1.591 6.602l4.339 2.76 7.494-7.493Z' />
        </svg>
      </div>
    </div>
  </div>
</template>

<script>
import { Client, Account, Databases, Query } from 'appwrite';

const client = new Client();
client
  .setEndpoint('http://localhost/v1') // Your Appwrite Endpoint
  .setProject('6380b9a5c3bf3275eb47');

const account = new Account(client);
const database = new Databases(client);

account.createAnonymousSession().then(
  (response) => {
    console.log(response);
  },
  (error) => {
    console.log(error);
  }
);

export default {
  name: 'IndexPage',
  data() {
    return {
      chatSessions: [],
      sessionID: '',
      messages: [],
      text: ''
    }
  },
  methods: {
    async getChatSessions() {
      let response = await database.listDocuments(
        '6380b9c55711dd457533',
        '6380d51354735c845c50',
      );

      this.chatSessions = response.documents;
    },
    async getChat(sessionID) {
      let response = await database.listDocuments(
        '6380b9c55711dd457533',
        '6380c352335119982dfa',
        [Query.equal('sessionID', sessionID)]
      );

      this.sessionID = sessionID;
      this.messages = response.documents;
    },
    async sendMessage() {
      await database.createDocument(
        '6380b9c55711dd457533',
        '6380c352335119982dfa',
        'unique()',
        {
          sessionID: this.sessionID,
          message: this.text,
          sender: 'admin',
        }
      );

      this.text = '';
    }
  },
  mounted() {
    this.getChatSessions();
    if (account.get !== null) {
      try {
        client.subscribe('documents', (response) => {
          this.getChat(this.sessionID)
        });
      } catch (error) {
        console.log(error, 'error');
      }
    }
  },
}
</script>

<style>
@import '../main.css';
</style>