FROM node:20

USER node

WORKDIR /home/node/code

# COPY --chown=node index.js .
# COPY --chown=node package.json .
# COPY --chown=node package-lock.json .
# becuase of .dockerignore we can do the following
COPY --chown=node . .

# special for building app on CI/CD pipelines and/or prod - it bases itself on the package-lock.json
RUN npm ci 

CMD ["node", "index.js"]
