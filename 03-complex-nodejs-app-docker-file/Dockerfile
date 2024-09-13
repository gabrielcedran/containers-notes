FROM node:20

USER node

WORKDIR /home/node/code


# we have to be smart how we build our images so we can benefit from cached layers (npm install will also be cached and only re-executed if the dependencies change)
COPY --chown=node package*.json .

# special for building app on CI/CD pipelines and/or prod - it bases itself on the package-lock.json
RUN npm ci 

# becuase of .dockerignore we can do the following
COPY --chown=node . .


CMD ["node", "index.js"]
