# Ollama

:::warning
Diese Komposition nutze ich nicht mehr, weshalb sie evtl. nicht mehr funktioniert.

:::

And another post about A.I… And why this headline? <a href="<https://chat.openai.com/>" target="_blank">OpenAI's ChatGPT</a>, <a href="<https://copilot.microsoft.com/>" target="_blank">Microsoft's Co-Pilot</a> and <a href="<https://www.salesforce.com/news/press-releases/2023/03/07/einstein-generative-ai/>" target="_blank">Salesforce's Einstein</a> are just running well! But, are they? And even if you are happy about them, are you willing to pay regular (not cheap) license fees for all of your employees, even if they just use it from time to time? Also, do you really trust Big Tech's promises about confidentiallity, when it's about your intellectual property? If you answer all of this by yes, you can stop reading, now. But, in case you would like to know how you easily can run your own A.I. chatbot or you are just curious like me, about how that is done, my article gives you an overview how to do that by utilizing the two amazing tools <a href="<https://ollama.com/>" target="_blank">Ollama</a> and <a href="<https://openwebui.com/>" target="_blank">Open WebUI</a>.

 ![](uploads/6cc33526-867e-42e8-9bcd-6c11598a0e64/7122add6-723d-447d-8016-f9f7e4aec689/01-a-chatbot.jpg " =2048x2048")

## Preparation and Requirements

Theoretically you could even run everything on a laptop, but for sure you would face several issues at least after a while. Better is to have a server with installed docker up and running. Also a reverse proxy, and an URL would make things more smooth, but are not mandatory. As we see later in chapter performance, it would be beneficial if your server has a dedicated graphics card, but also that is not a must.

## Installation and Configuration

The guys from the „Open WebUI" project made it extremely easy to get your chatbot running. Basically you just create a new docker-compose.yml file like the one in the example below and start the thing as usual by command „docker compose up -d". That's it, no joke!

```yaml

services:
  chat:
    container_name: chat
    image: ghcr.io/open-webui/open-webui:ollama
    volumes:
      - ./ollama:/root/.ollama
      - ./open-webui:/app/backend/data
    restart: unless-stopped
    #ports:
    #  - 8080:8080
    networks:
      caddy:
networks:
  caddy:
    external: true
```

As you can see in my example file I customized the network configuration, and also configured my reverse proxy caddy to point access to [chat.handtrixxx.com](http://chat.handtrixxx.com) to my new container. As you can see in the following screenshot you can click on „Sign up" to create a new user account for yourself as Administrator.

 ![](uploads/6cc33526-867e-42e8-9bcd-6c11598a0e64/6a0836b0-af33-4126-8959-9e255afee4c7/02-grafik-1.png " =731x531")


Now, after you logged in, there are just two steps more to do to start your A.I. chats. At first you should go to the admin panel and then at the „Admin settings" to disable registration for other users to avoid other users just create an account on your instance. Then in the settings at the models tab you will have to download one ore more language models. There are plenty to choose from. An overview is available at: <https://ollama.com/library> . You are done and as you see it does not have to take more than 5 minutes in case you are a bit experienced in docker and setting up tools in general.

## Costs

Since everything I introduced and described is based on Open Source software, there are no costs or licensing fees at all. Great, isn't it? But to say it is completly free is also not completly true, since you have to cover the charges for the server if you do not „have" one anyway 🙂.

## Performance

As mentioned before, a dedicated graphics card would speed up the response times of the chatbot trendemously. By running it only on CPU, like i did in my example, every generation of a response took all the CPU power i have (and I have a lot) for some seconds. So the whole thing feels a bit like the early versions of ChatGPT. That's no drama, but definitly noticeable.

## Conclusion

As conclusion i let the openchat language model answer itself to my prompt:

 ![](uploads/6cc33526-867e-42e8-9bcd-6c11598a0e64/55de7a21-58b9-4b79-b5d2-898a280ea91c/03-grafik-2.png " =838x611")