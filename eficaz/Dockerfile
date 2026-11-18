# syntax=docker/dockerfile:1
# Imagem da Calculadora RTC — Simples Nacional (Receita Federal, MIT), construída pela Eficaz.
# Contexto do build = checkout da tag rtc-<versão> do Simples.
# Contexto nomeado "regular" = checkout da tag do regime regular, de onde sai o banco
# (as 7 tabelas que o Simples lê são criadas pelo flyway do regular: V0049/V0050).

FROM eclipse-temurin:21-jdk AS banco
WORKDIR /regular
COPY --from=regular . .
RUN chmod +x mvnw \
 && mkdir -p calculadora/db \
 && ./mvnw -B -ntp flyway:migrate -Dflyway.configFiles=./flyway/flyway-pro.conf \
 && test -s calculadora/db/calculadora-pro.db

FROM eclipse-temurin:21-jdk AS build
WORKDIR /src
COPY . .
RUN chmod +x mvnw \
 && ./mvnw -B -ntp clean package -DskipTests \
 && test -s target/api-simples-nacional.jar

FROM eclipse-temurin:21-jre
ARG REVISION=desconhecido
ARG VERSION=desconhecida
ARG DB_REVISION=desconhecido
LABEL org.opencontainers.image.title="rtc-simples" \
      org.opencontainers.image.description="Calculadora de Tributos da Reforma (CBS/IBS) — Simples Nacional, perfil offline" \
      org.opencontainers.image.source="https://github.com/carlosallef/calculadora-rtc-simples-nacional" \
      org.opencontainers.image.licenses="MIT" \
      org.opencontainers.image.revision="${REVISION}" \
      org.opencontainers.image.version="${VERSION}" \
      br.com.eficaz.rtc.db.revision="${DB_REVISION}"
RUN apt-get update \
 && apt-get install -y --no-install-recommends curl \
 && rm -rf /var/lib/apt/lists/* \
 && useradd --system --uid 10001 --home /app rtc
WORKDIR /app
COPY --from=build /src/target/api-simples-nacional.jar app.jar
COPY --from=banco /regular/calculadora/db/calculadora-pro.db calculadora/db/calculadora-pro.db
COPY --from=build /src/LICENSE LICENSE
USER rtc
ENV TZ=America/Sao_Paulo \
    JAVA_OPTS="-Xmx640m -XX:+ExitOnOutOfMemoryError"
EXPOSE 8082 9103
HEALTHCHECK --interval=30s --timeout=5s --start-period=90s --retries=3 \
  CMD curl -fsS http://localhost:9103/health || exit 1
# O jar roda de /app, que contém ./calculadora/db. DB_DOWNLOAD_URL fica vazio de propósito (I2).
ENTRYPOINT ["sh", "-c", "exec java $JAVA_OPTS -Dspring.profiles.active=offline -jar app.jar"]
