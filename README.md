# monolith-ui

> This is the front end of monolith-mail 's anonymous post office, used to display incoming mail

***

Experience the address:http://8.136.23.192:9091/#/

***
Used in conjunction with the back end...<br/>
Project backend address:https://github.com/linenhui023/monolith-mail

***

## docker type install (recommended)
- 1:set config/index.js URL address and port, pointing to the Java project
- 2:build image
- 3:exec run command
``` docker
docker run --name <name> -d -p port:port monolith-ui:<tag>
```
***
## npm type install

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report

# run e2e tests
npm run e2e

# run all tests
npm test
```
***
> Made a simple front end UI, will slowly modify the front end interface code。<br>
> The main reason is that my thoughts on the front end are still stuck in the era of jQuery. I temporarily learned Vue and elementUI components and made such a simple display interface
