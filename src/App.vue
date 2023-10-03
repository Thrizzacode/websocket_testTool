<template>
  <q-layout view="hHh lpR fFf">
    <q-header elevated class="bg-primary text-white">
      <q-toolbar>
        <q-toolbar-title> WebSocket測試網 </q-toolbar-title>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <q-page padding>
        <div class="space-x-4">
          <q-btn color="primary" label="link start!" @click="startSignalRConnection" />
          <q-btn color="primary" label="發送Invoke" @click="sendInvoke" />
          <q-btn color="primary" label="斷開魂結" @click="stopConnection" />
          <q-btn color="primary" label="萬物皆空" @click="clear" />
        </div>
        <q-input v-model="signalRUrl" type="text" label="請輸入signalR的Url" />
        <q-input v-model="invokeValue" type="text" label="請輸入invoke的值" />
        <q-input v-model="invokeParam" type="text" label="請輸入invoke的參數" />
        <q-input v-model="onValue" type="text" label="請輸入on的值" />
        <ul>
          <li v-for="(message, index) in messages" :key="index">{{ index + 1 }}: {{ message }}</li>
        </ul>
      </q-page>
    </q-page-container>
  </q-layout>
</template>
<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue';
import * as signalR from '@microsoft/signalr';
import { useQuasar } from 'quasar';

const $q = useQuasar();
const signalRUrl = ref('https://localhost:7193/clientHub');
const invokeValue = ref('InvokeBaccaratGameLobby');
const invokeParam = ref('');
const onValue = ref('ReceiveLobbyData');
const messages = ref([]);
let connection;

//建立SignalR連線
const createConnection = () => {
  connection = new signalR.HubConnectionBuilder()
    .withUrl(signalRUrl.value, {
      skipNegotiation: true,
      transport: signalR.HttpTransportType.WebSockets
    })
    .build();
};

//開始SignalR連線
const startSignalRConnection = async () => {
  try {
    createConnection();
    await connection.start();
    $q.notify({ position: 'top', color: 'positive', message: 'SignalR Connected.' });
  } catch (error) {
    console.error('SignalR Connection Error: ', error);
  }
};

//發送Invoke
const sendInvoke = () => {
  connection
    .invoke(invokeValue.value, invokeParam.value)
    .then(() => {
      connection.on(onValue.value, (data) => {
        console.log('ReceiveData', data);
        clear();
        messages.value.push(data);
      });
    })
    .catch((error) => {
      console.error(error);
      $q.notify({ position: 'top', color: 'negative', message: error.toString() });
    });
};

//斷開連線
const stopConnection = () => {
  connection.stop();
  $q.notify({ position: 'top', color: 'negative', message: 'SignalR Disconnected.' });
};

const clear = () => {
  messages.value = [];
};
</script>
<style lang="scss" scoped></style>
