FROM c2labs.azurecr.io/atlas:dev

WORKDIR /app
COPY wait-for-it.sh .
ENTRYPOINT ["./Atlas"]
# ENTRYPOINT ["./wait-for-it.sh", "atlas-db:1433", "--timeout=15", "--strict", "--", "./Atlas"]