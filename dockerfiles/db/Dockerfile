ARG PG_VERSION=15
FROM twentycrm/twenty-postgres:latest as build
USER root
RUN apt-get update 

ENV build_deps ca-certificates \
  git \
  build-essential \
  libpq-dev \
  curl \
  libreadline6-dev \
  zlib1g-dev \
  lsb-release


RUN apt-get install -y --no-install-recommends $build_deps pkg-config cmake

RUN install -d /usr/share/postgresql-common/pgdg && \
    curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc && \
    sh -c 'echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && \
    apt-get update && \
    apt-get install -y postgresql-server-dev-15

WORKDIR /home/pg_graphql

ENV HOME=/home/pg_graphql \
  PATH=/home/pg_graphql/.cargo/bin:$PATH
RUN chown postgres:postgres /home/pg_graphql
USER postgres

RUN \
  curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --no-modify-path --profile minimal --default-toolchain stable && \
  rustup --version && \
  rustc --version && \
  cargo --version

# PGRX
RUN cargo install cargo-pgrx --version 0.12.6 --locked

RUN cargo pgrx init --pg15 $(which pg_config)

USER root

COPY . .
RUN cargo pgrx install

RUN chown -R postgres:postgres /opt/bitnami/postgresql && \
    chown -R postgres:postgres /bitnami/postgresql
    
USER postgres

FROM twentycrm/twenty-postgres:latest as release

USER root
COPY --from=build /opt/bitnami/postgresql/share/extension/pg_graphql*.* /opt/bitnami/postgresql/share/extension/
COPY --from=build /opt/bitnami/postgresql/lib/pg_graphql.so /opt/bitnami/postgresql/lib/
