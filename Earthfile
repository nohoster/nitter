VERSION 0.7
FROM alpine:3.18
WORKDIR /src/

base-build:
  BUILD \
        --platform=linux/amd64 \
        --platform=linux/arm64 \
        +base-img

deps:
  RUN apk --no-cache add gcc git libc-dev libsass-dev nim nimble pcre
  COPY nitter.nimble config.nims .
  RUN nimble install -y --depsOnly
 
build:
  FROM +deps
  COPY --dir src public tools .
  RUN nimble build -d:danger -d:lto -d:strip && nimble scss && nimble md
  SAVE ARTIFACT nitter
  SAVE ARTIFACT public/ 

base-img:
  COPY +build/nitter .
  COPY +build/public public/
  ENTRYPOINT ["./nitter"]
  SAVE IMAGE --push repo.nohost.network/nitter:base

prod-build:
  FROM repo.nohost.network/nitter:base
  RUN apk --no-cache add ca-certificates pcre openssl1.1-compat curl jq
  RUN --secret GUEST_TOKENS_TOKEN --push curl "https://twitterminator.x86-64-unknown-linux-gnu.zip/download?key=$GUEST_TOKENS_TOKEN&host=nitter.nohost.network" | jq -n '[inputs]' > guest_accounts.json
  ENTRYPOINT ["./nitter"]
  SAVE IMAGE --push repo.nohost.network/nitter:latest

prod-push:
   BUILD \
        --platform=linux/amd64 \
        --platform=linux/arm64 \
        +prod-build 
