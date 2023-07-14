# companion-app.pinokio

A one-click installer for https://github.com/a16z-infra/companion-app powered by [Pinokio](https://pinokio.computer/)

![companion.gif](companion.gif)

# How the script works

## install.json

```json
{
  "run": [{
    "method": "shell.run",
    "params": {
      "message": "git clone https://github.com/a16z-infra/companion-app.git"
    }
  }, {
    "method": "shell.run",
    "params": {
      "message": "npm install",
      "path": "companion-app"
    }
  }, {
    "method": "input",
    "params": {
      "title": "Settings",
      "form": [{
        "title": "OpenAI API KEY",
        "key": "OPENAI_API_KEY",
        "description": "Find your OpenAI API key <a target='_blank' href= 'https://platform.openai.com/account/api-keys'>here</a>"
      }, {
        "title": "Pinecone Index Name",
        "key": "PINECONE_INDEX",
        "description": "Make sure to set up pinecone by following the instructions <a target='_blank' href= 'https://github.com/a16z-infra/companion-app#3-fill-out-secrets'>here</a>"
      }, {
        "title": "Pinecone Index Environment",
        "key": "PINECONE_ENVIRONMENT"
      }, {
        "title": "Pinecone API KEY",
        "key": "PINECONE_API_KEY"
      }, {
        "title": "NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY",
        "key": "NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY",
        "description": "Set up clerk by following the instructions <a target='_blank' href='https://github.com/a16z-infra/companion-app#3-fill-out-secrets'>here</a>)"
      }, {
        "title": "CLERK_SECRET_KEY",
        "key": "CLERK_SECRET_KEY"
      }, {
        "title": "UPSTASH_REDIS_REST_URL",
        "key": "UPSTASH_REDIS_REST_URL",
        "description": "Set up Upstash by following the instructions <a target='_blank' href='https://github.com/a16z-infra/companion-app#3-fill-out-secrets'>here</a> and scroll down to the 'Upstash API key' part)"
      }, {
        "title": "UPSTASH_REDIS_REST_TOKEN",
        "key": "UPSTASH_REDIS_REST_TOKEN"
      }]
    }
  }, {
    "method": "fs.write",
    "params": {
      "path": "companion-app/.env.local",
      "text": [
        "VECTOR_DB=pinecone",
        "",
        "NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY={{input.NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY}}",
        "CLERK_SECRET_KEY={{input.CLERK_SECRET_KEY}}",
        "NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in",
        "NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up",
        "NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/",
        "NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/",
        "",
        "OPENAI_API_KEY={{input.OPENAI_API_KEY}}",
        "",
        "PINECONE_API_KEY={{input.PINECONE_API_KEY}}",
        "PINECONE_ENVIRONMENT={{input.PINECONE_ENVIRONMENT}}",
        "PINECONE_INDEX={{input.PINECONE_INDEX}}",
        "",
        "UPSTASH_REDIS_REST_URL={{input.UPSTASH_REDIS_REST_URL}}",
        "UPSTASH_REDIS_REST_TOKEN={{input.UPSTASH_REDIS_REST_TOKEN}}"
      ],
      "join": "{{os.EOL}}"
    }
  }, {
    "method": "shell.run",
    "params": {
      "message": "npm run generate-embeddings-pinecone",
      "path": "companion-app/companions"
    }
  }, {
    "method": "notify",
    "params": {
      "html": "<b>Install complete</b><br><br>Click here to go to the start script!",
      "href": "./start.json"
    }
  }]
}
```

1. The first step runs `shell.run` with `git clone https://github.com/a16z-infra/companion-app.git", which clones the companion-app github repository
2. The second command runs `npm install`
3. The third takes a user input. This is where you enter the API keys from all the required API providers such as OpenAI, Pinecone, Clerk, Upstash, etc.
4. The input is passed onto the next action `fs.write` as the variable `input`
5. The `fs.write` API creates a file named `companion-app/.env.local` with the text inside the array. Note that it's using the template expressions with the `input` variable passed in from the previous step.
6. After the environment file is written, it runs `npm run generate-embeddings-pinecone` inside the `companion-app/companions` folder
7. After all this is finished it displays a notification pop up, which when clicked sends you to the [start.json](start.json) script.


## start.json

```json
{
  "run": [{
    "method": "shell.start",
    "params": {
      "path": "companion-app"
    }
  }, {
    "method": "shell.enter",
    "params": {
      "message": "npm run dev",
      "on": [{
        "event": "/(http:\\S.+)/",
        "return": "{{event.matches[0][1]}}"
      }]
    },
    "notify": true
  }, {
    "method": "browser.open",
    "params": {
      "uri": "{{input}}",
      "target": "_blank"
    }
  }, {
    "method": "process.wait"
  }]
}
```

1. At first it creates a shell with the default execution path of `companion-app` (Every subsequent command that runs through `shell.enter` will be executed from this path)
2. Then we run the command `npm run dev` with the `shell.enter` call.
3. The `shell.enter` doesn't return until the terminal encounters a pattern that matches the `/(https:/\\S.+)/` pattern. When it does, The match object is stored in `event.matches`, and we can return the exact match we want to the next action.
4. Finally, since we now know that the server has started, we open the browser using the http URL passed in from the previous step.
5. Don't forget to end with `process.wait` because, if you don't include this step, all the processes that started with this script will automatically shut down and garbage collect once this script has finished. By adding the `process.wait` call, we are ensuring that this script never ends, and goes into a standby mode.
