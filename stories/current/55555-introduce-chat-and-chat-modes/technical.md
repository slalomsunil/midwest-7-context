
What We're Building
1. allow user to select an online user to chat
2. user then selects a chat mode, user types a message
3. the message is transformed into a personality and emojis based on the mode
4. the receiver responds back the same way.

Technical Approach


Implementation Steps

[Step 1 - what you'll do first]
[Step 2 - next action]
[Step 3 - and so on]


Data Changes
1. save the chat data in the db and clear it when user both users go offline. Do not clear it when only one user goes offline as the other person still continues to see it.

API Changes (if applicable)
new api to save the chats and update chat thread continuously from both users in the chat