FROM ethereum/client-go

WORKDIR /home/ethereum

COPY geth/static-nodes.json* /home/ethereum/
COPY pwd /home/ethereum/pwd
COPY keystore /home/ethereum/keystore
COPY geth /home/ethereum/geth

ENTRYPOINT []

EXPOSE 8501 30311 30311/udp
