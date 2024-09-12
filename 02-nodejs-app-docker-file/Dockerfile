FROM node:20

RUN useradd -ms /bin/bash anotheruser

USER anotheruser

COPY --chown=anotheruser  index.js /home/anotheruser/index.js 

CMD ["node", "/home/anotheruser/index.js"]