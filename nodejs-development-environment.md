[🏠](README.md) › [Node.js](nodejs.md) › Development environment


# Development environment

- [Docker](#docker)
- [IDE](#ide)
- [Linting](#linting)
- [OSX Programs](#osx-programs)
- [Version control](#version-control)


## Docker

- Microservices often defer tasks to one another.  
- This includes repos with the names `service-*`, `edge-*`, `worker-*` & `web-*`.  
- Developing these repos in isolation requires other services to be running.  
- Docker allows us to:  
  - run multiple services with a single command.  
  - develop on same platform (linux) with same technology (docker) as production.  
  - aggregate logs into a single shell.  
  - No configuration of local development machine (i.e. hosts files), just requires Docker for Mac to be installed.  
- Note: Windows has issues with setting perms on private keys & interactive mode for docker-compose.  


### Setup

**1. Create .env file with KEY and PROJECT_PATH**  
`cp .env.sample .env`

**1. Create npm volume**  
`docker volume create --name npm`

**1. Install dependencies**  
`npm run docker -- npm install`

**1. Run cluster**  
This will run a local copy of `edge-account` and `service-account`.  
`npm run cluster`


**Available commands**  

```bash

export KEY=~/.ssh/gritcode_rsa
export PROJECT_PATH=~/projects/gritcode

# test app
npm run docker -- npm test
# install dependencies
npm run docker -- npm install
# run single app with debugging tools
npm run docker -- npm debug
# run single app
npm run docker -- npm start
# run app cluster
npm run cluster
```


## IDE
- atom.io

**Plugins**
  - atom-ternjs (javascript autocompletion)
  - linter
  - linter-eslint
  - project-manager
  - file-icons


## linting
- Linting is neccessary for enforcing a developer agnostic coding style standard and for catching syntax errors before runtime.
- We extend the [airbnb javascript styleguide](https://github.com/airbnb/javascript) for linting.  

~/.eslintrc
```javascript
{
  "parser": "babel-eslint",
  "extends": "airbnb/base",
  "plugins": [
    "react"
  ],
  "env": {
    "node": true
  },
  "rules": {
    "no-console": 0,
    "no-param-reassign": 0,
    "max-len": 0,
    "padded-blocks": 0,
  }
}
```

**additional reading**  
[JSJ ESLint with Jamund Ferguson](https://devchat.tv/js-jabber/162-jsj-eslint-with-jamund-ferguson)


## OSX Programs

### GUI apps

- [Redis](https://github.com/luin/medis)
- [Postgres](https://github.com/sosedoff/pgweb)
- [Node.js debugging](https://github.com/Jam3/devtool)
- [Git](https://www.sourcetreeapp.com)

### CLI tools

- [Userland package manager](http://brew.sh)
- [Docker](https://github.com/nlf/dlite)


## Version control
- Use git.
- Avoid personal repos. Commit your source code to Gitlab.

### gitflow
- Use Gitflow for branch management
- Coming soon instructions on SourceTree or automatating branch management

**additional reading**  
- [Git extensions for gitflow](https://github.com/nvie/gitflow)
