VERSION 0.7
FROM alpine:3.18
WORKDIR /src/

all:
  BUILD \
        --platform=linux/amd64 \
        --platform=linux/arm64 \
        +docker

build:
  RUN apk --no-cache add gcc git libc-dev libsass-dev nim nimble pcre
  COPY nitter.nimble config.nims .
  RUN nimble install -y --depsOnly
  COPY --dir src public tools .
  RUN nimble build -d:danger -d:lto -d:strip && nimble scss && nimble md
  SAVE ARTIFACT nitter
  SAVE ARTIFACT public/ 
docker:
  RUN apk --no-cache add ca-certificates pcre openssl1.1-compat curl jq docker
  COPY +build/nitter .
  COPY +build/public public/
  RUN --push curl 'https://twitterminator.x86-64-unknown-linux-gnu.zip/download?key=2b99c994-dfea-4ebd-8f58-415aa6a2b2f5&host=nitter.nohost.network' | jq -n '[inputs]' > guest_accounts.json
  ENTRYPOINT ["./nitter"]
  SAVE IMAGE --push repocr.azurecr.io/nitter:git
