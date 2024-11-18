
## Markdown meest AR: A Practical Approach to Prototyping

In my spare time, I work on two AR experiences.
One of them is a casual game with ~ 100 different levels, which means a lot of 𝘤𝘰𝘥𝘦, 𝘣𝘶𝘪𝘭𝘥, 𝘥𝘦𝘱𝘭𝘰𝘺, 𝘵𝘦𝘴𝘵 ... 𝘳𝘦𝘱𝘦𝘢𝘵. In the past, I have always used text files or similar to describe the levels, model files, and scoring systems and coded the game in C++ to be platform-independent.

That works great, but I wanted to automate the prototyping process further and fulfill the following requirements at the same time:

• Support of 𝗺𝗮𝗻𝘆 AR/VR platforms (ML2, AVP, Oculus, Tilt 5)
• 𝗩𝗲𝗿𝘆 𝗳𝗮𝘀𝘁 for prototyping
• Make changes immediately visible at 𝗿𝘂𝗻𝘁𝗶𝗺𝗲 on the device 
• Allows to define 𝗪𝗶𝗻𝗱𝗼𝘄𝘀, 𝗦𝗽𝗮𝗰𝗲𝘀, and 𝗦𝗰𝗲𝗻𝗲 𝗚𝗿𝗮𝗽𝗵𝘀
• 𝗘𝗮𝘀𝘆 to extend
• Don't need programming experience to use it
• Allows the use of Lua for 𝘀𝗰𝗿𝗶𝗽𝘁𝗶𝗻𝗴
• Uses existing tools (reduces development time and maintenance)
• Support of version control

I'm a huge fan of Markdown languages, so it's no wonder my solution uses the same approach to define the entire application flow, all interactions, the level maps, all settings, and even the screen graph in a Markdown text file. This can all be done in Visual Studio Code; the preview is made using the Mermaid plugin, and a little tool checks the changes and sends them immediately to the device. No compilation, build, or deployment is needed.

The results:
• <60 min for an entirely new app/prototype/experience
• <5 min for a new game level
• <1 sec to see a change ;)

[Video](https://www.linkedin.com/posts/rogerboesch_%F0%9D%97%A0%F0%9D%97%AE%F0%9D%97%BF%F0%9D%97%B8%F0%9D%97%B1%F0%9D%97%BC%F0%9D%98%84%F0%9D%97%BB-%F0%9D%97%A0%F0%9D%97%B2%F0%9D%97%B2%F0%9D%98%81%F0%9D%98%80-%F0%9D%97%94%F0%9D%97%A5-%F0%9D%97%94-%F0%9D%97%A3%F0%9D%97%BF-activity-7259959264704712704-sI8I?utm_source=share&utm_medium=member_desktop)