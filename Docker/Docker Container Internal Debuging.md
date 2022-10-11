### Docker Container Internal Debuging

### ["curl: not found"](https://stackoverflow.com/questions/64300599/why-do-i-get-curl-not-found-inside-my-nodealpine-docker-container)
- [Issue]
    ```
    /usr/app # curl naver.com
    sh: curl: not found
    ```
- [Solution]
    ```
    apk --no-cache add curl
    ```