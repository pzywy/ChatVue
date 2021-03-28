<template lang='pug'>
.view.login(v-if="state.username ==='' || state.username === null")
  form.login-form(@submit.prevent="Login")
    h1 Login to chat
    label(for="username") Username:
    input(type="text" placeholder="Enter your name..." v-model="inputUsername")
    input(type="submit" value="Login")

.view.chat(v-else)
  header
    h1 Welcome, {{state.username}}
    button.logout(@click="Logout") Logout
  section.chat-box
    div(v-for="message in state.messages" :key="message.key" :class="(message.username == state.username ? 'message current-user' : 'message')")
      div.username {{message.username}}
      div.content {{message.content}}
  footer
    form.send-message(@submit.prevent="SendMessages")
      input(type="text" v-model="inputMessages" placeholder="Write a message...")
      input(type="submit" value="Send")

</template>

<script>
import { reactive, onMounted, ref } from "vue";
import db from "./db";


export default {
  setup()
  {
    const inputUsername = ref("");
    const inputMessages = ref("");

    const state = reactive({
      username:"",
      messages: []
    })

    const Login = () => {
      if(inputUsername.value != "" || inputUsername.value != null){
        state.username = inputUsername.value;
        inputUsername.value = ""; 
        localStorage.setItem('username', state.username);
      } 
    }

    const SendMessages = () => {
      
      const messagesRef = db.database().ref("messages");

      if(inputMessages.value === "" || inputMessages.value === null) return;

      const message = {
        username: state.username,
        content: inputMessages.value
      }

      try{
        messagesRef.push(message);
      }catch(error){
        console.error(error);
      }
      finally{
        console.log("Message Sent Succesfuly!")
      }
      
      inputMessages.value="";
    }

    onMounted( () => {
      //###########CHECK LOCALSTORAGE FOR RECENTLY LOGGED USER#############
      
      if(localStorage.getItem('username'))
      {
        inputUsername.value = localStorage.getItem('username');
        Login();
      }

      //##########LOAD MESSAGES###############
      const messagesRef = db.database().ref("messages");

      messagesRef.on("value", snapshot => {
        const data = snapshot.val();
        let messages = [];

        Object.keys(data).forEach(key => {
          messages.push({
            id: key,
            username: data[key].username,
            content: data[key].content
          });
        });
        state.messages = messages;
      });
    });

    const Logout = () => {
      state.username = "";
      localStorage.removeItem('username');
    }

    return {
      inputUsername,
      inputMessages,
      Login,
      SendMessages,
      state,
      Logout
    }
  }
}
</script>

<style lang="scss">
$backgroundColor:#ffffea;

$chatBackgroundColor:#9bc1bc;
$UserTextColor:#00cecb;
$OtherTextColor:#ffed66;
$textColor:black;
$authorColor:rgb(219, 219, 219);

$headerColor: #ff5e5b;

$loginBackgroundColor:#00cecb;
$loginTextColor:white;
$loginSubmitColor:#ffed66;
body
{
  margin: 0 !important;
  padding: 0 !important;
  background-color: $backgroundColor;
}
.login
{

  width:75vw;
  max-width: 30em;
  height: auto;
  padding: 2em 0.5em;
  justify-content: center;
  text-align: center;
  margin:15vh auto;
  
  box-shadow: 2px 2px 5px 3px black;

  background-color: $loginBackgroundColor;
  color:$loginTextColor;
  border-radius: .5em;
  * 
  {
    display:flex;
    flex-direction: column;
    align-items: center;
  }
  input
  {
    width:50%;
    margin-top: .5em;
    border-radius: .3em;
    &[type=submit]
    {
      background-color: $loginSubmitColor;
      &:hover
      {
      cursor: pointer;
      }
    }
  }
  
  :focus
  {
    outline: none;
  }
}

header
{
  
  background-color: $headerColor;
  padding: 1em;
  margin:0;
  *
  {
    margin-top: 1em;
    margin:auto;
  }
}
.logout
{
  background-color: red;
  color: white;
  font-size: 1.2em;
  border-radius: .5em;
  
  height: fit-content;
  position: absolute;
  right: 1em;
  top:1em;
  border:white solid 2px;
  &:hover{
      cursor: pointer;
    }
  &:focus
  {
    outline: none;
  }
  
}

.chat-box
{
  height: 70vh;
  width: 95vw;
  background-color:$chatBackgroundColor;
  border-radius: .5em;
  margin: 1em auto;
  padding-top: 1em;
  overflow-y: scroll;
}

footer
{
  width: 100%;
  display: flex;
  justify-content: center;
  * 
  {
    font-size: 1.5em;
  }


}



.message 
{
  
  .content
  {
  background-color: $OtherTextColor;
  border-radius: .5em;
  color:$textColor;
  text-align: center;
  padding:.4em;
  }
  
  margin: .2em;
  width: 60%;
  height: min-content;
  padding: .2em;

  margin-left: 1em;
  margin-right: auto;
  
  
  .username
  {
    text-align: left;
    background-color: $authorColor;
    width: min-content;
    margin-left: 0;
    margin-right: auto;
    padding: .2em;
    margin-bottom: .2em;
    border-radius: .5em;
  }
  &.current-user 
  {
    margin-right: 1em;
    margin-left: auto;
    .username
    {
    margin-right: 0;
    margin-left: auto;
    }
    .content
    {
      
      background-color: $UserTextColor;
    }
    
  }
}

.send-message
{
  width:95vw;
  margin: auto;
  font-size: 1rem;
  text-align-last: center;
  input[type="text"]
  {
    border-radius: .5em;
    border:black solid 2px;
    width: auto;
    max-width: 60vw;
  }
  input[type="submit"]
  {
    width: auto;
    margin-left:.5em;
    border-radius: 1em;
    border:black solid 1px;
    &:hover{
      cursor: pointer;
    }
  }
  input:focus
  {
    outline: none;
  }
}
</style>
