# Use uma imagem oficial do Java runtime como imagem pai
FROM openjdk:8u181-jdk-alpine

# Disponibilize a porta 25565 para fora do container
EXPOSE 25565

ADD server.jar server.jar
RUN echo "eula=true" > eula.txt

# Execute o servidor Minecraft
CMD java -Xmx1024M -Xms1024M -jar server.jar nogui