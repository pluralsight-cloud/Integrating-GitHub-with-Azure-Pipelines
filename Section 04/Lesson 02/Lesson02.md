Create code directory:
```
mkdir code
```

Move code into code directory:
```
mv !(code) code
```

Create Dockerfile:
```
FROM node:16-alpine AS deps

WORKDIR /app
COPY code/ .
RUN yarn install --frozen-lockfile

FROM node:16-alpine AS BUILD_IMAGE

WORKDIR /app

COPY --from=deps /app/ ./
COPY . .
RUN yarn build

RUN rm -rf node_modules
RUN yarn install --production --frozen-lockfile --ignore-scripts --prefer-offline

FROM node:16-alpine

ENV NODE_ENV production

RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

WORKDIR /app
COPY --from=BUILD_IMAGE --chown=nextjs:nodejs /app/package.json /app/yarn.lock ./
COPY --from=BUILD_IMAGE --chown=nextjs:nodejs /app/node_modules ./node_modules
COPY --from=BUILD_IMAGE --chown=nextjs:nodejs /app/public ./public
COPY --from=BUILD_IMAGE --chown=nextjs:nodejs /app/.next ./.next

USER nextjs

EXPOSE 3000

CMD [ "yarn", "start" ]
```

Test the image build:
```
sudo docker image build -t helios-proof:latest .
```
