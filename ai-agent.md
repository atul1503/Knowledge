To create docs for any tech, use vs code agent or cursor agent by providing following prompt:

```
Look at this folder how I document concepts of different tech, analyze it and create a similar [your tech] md file.
Use similar language simplicity and style. Explain each new property or field or functionality the very first time you use in any example.
For eg, look at dart, kotlin and swift docs here.
```
Copy this into agent mode chat and say "here [your tech] is react native" etc.


To modify or add something use this prompt:

```
Look at this folder how I document tech concepts. Analyze this language simplicity, tone and organization to [your changes].
```

Don't use the same chat for more than 4 times. Start a new chat after 4 prompts.
Give small tasks for each prompt.

To modify or add something(another and more simpler way):

Go the file click on an empty line where you want it to add something and click [command+k] and enter the prompt in agent mode. Prompt should have "document" word in it instead of "explain:

```
Document why is this required.
```