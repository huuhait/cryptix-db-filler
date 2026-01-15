FROM python:3.10-alpine

ARG REPO_DIR

EXPOSE 8000

ENV CRYPTIXD_HOST_1=n.seeder1.cryptixd.net:16110
ENV SQL_URI=postgresql+asyncpg://postgres:password@postgresql:5432/postgres
ARG version
ENV VERSION=$version

RUN apk --no-cache add \
  git \
  gcc \
  libc-dev \
  build-base \
  linux-headers \
  libpq-dev \
  dumb-init

RUN pip install \
  pipenv \
  grpcio-tools==1.63.0

RUN addgroup -S -g 55746 api \
  && adduser -h /app -S -D -g '' -G api -u 55746 api

WORKDIR /app

USER api

COPY --chown=api:api . /app

# Generate protobuf files and fix imports
RUN cd cryptixd && python -m grpc_tools.protoc \
    -I./protos \
    --python_out=. \
    --grpc_python_out=. \
    ./protos/rpc.proto ./protos/messages.proto ./protos/p2p.proto \
    && sed -i 's/^import rpc_pb2/from . import rpc_pb2/' messages_pb2.py \
    && sed -i 's/^import p2p_pb2/from . import p2p_pb2/' messages_pb2.py \
    && sed -i 's/^import messages_pb2/from . import messages_pb2/' messages_pb2_grpc.py

RUN pipenv install

ENTRYPOINT ["/usr/bin/dumb-init", "--"]

CMD pipenv run python main.py
