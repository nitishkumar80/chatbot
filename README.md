## Chatbot-react-native




import React, { useState } from 'react';
import { View, TextInput, Text, FlatList, StyleSheet, TouchableOpacity } from 'react-native';

function Chatbot() {
  const [input, setInput] = useState('');
  const [messages, setMessages] = useState([
    {
      message: 'Hello I am ChatbotT',
      sender: 'ChatGPT',
    },
  ]);

  const handleChange = (text) => {
    setInput(text);
  };

  const handleSend = async () => {
    const newMessage = {
      message: input,
      sender: 'user',
    };

    const newMessages = [...messages, newMessage];

    setMessages(newMessages);

    setInput('');

    await processMessageToChatGPT(newMessages);
  };

  async function processMessageToChatGPT(chatMessages) {
    const API_KEY = 'sk--------';
    let apiMessages = chatMessages.map((messageObject) => {
      let role = '';
      if (messageObject.sender === 'ChatGPT') {
        role = 'assistant';
      } else {
        role = 'user';
      }
      return { role: role, content: messageObject.message };
    });
  
    const systemMessage = {
      role: 'system',
      content: 'Explain all concepts like I am 10 years old',
    };
  
    const apiRequestBody = {
      model: 'gpt-3.5-turbo',
      messages: [systemMessage, ...apiMessages],
    };
  
    try {
      const response = await fetch('https://api.openai.com/v1/chat/completions', {
        method: 'POST',
        headers: {
          Authorization: `Bearer ${API_KEY}`,
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(apiRequestBody),
      });
  
      const data = await response.json();
  
      if (response.ok) {
        // Check if data has the expected structure
        if (data.choices && data.choices.length > 0 && data.choices[0].message && data.choices[0].message.content) {
          console.log(data.choices[0].message.content);
  
          setMessages([
            ...chatMessages,
            {
              message: data.choices[0].message.content,
              sender: 'ChatGPT',
            },
          ]);
        } else {
          console.error('Invalid response from OpenAI API:', data);
        }
      } else {
        console.error('OpenAI API request failed with status:', response.status);
        console.error('Error details:', data.error);
      }
    } catch (error) {
      console.error('Error fetching data:', error);
    }
  }
  

  return (
    <View style={styles.container}>
      <View style={styles.responseArea}>
        <FlatList
          data={messages}
          keyExtractor={(item, index) => index.toString()}
          renderItem={({ item }) => (
            <View style={item.sender === 'ChatGPT' ? styles.gptMessage : styles.userMessage}>
              <Text>{item.message}</Text>
            </View>
          )}
        />
      </View>
      <View style={styles.promptArea}>
        <TextInput
          style={styles.input}
          placeholder="Send a message..."
          value={input}
          onChangeText={handleChange}
        />
        <TouchableOpacity style={styles.submitButton} onPress={handleSend}>
          <Text>Send</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  responseArea: {
    flex: 1,
  },
  gptMessage: {
    backgroundColor: 'lightblue',
    padding: 10,
    margin: 5,
    borderRadius: 8,
  },
  userMessage: {
    backgroundColor: 'lightgreen',
    padding: 10,
    margin: 5,
    borderRadius: 8,
  },
  promptArea: {
    flexDirection: 'row',
    alignItems: 'center',
    padding: 10,
  },
  input: {
    flex: 1,
    borderWidth: 1,
    borderColor: 'gray',
    height: 40,
    marginRight: 10,
    padding: 10,
    borderRadius: 8,
  },
  submitButton: {
    backgroundColor: 'yellow',
    padding: 10,
    borderRadius: 8,
  },
});

export default Chatbot;
