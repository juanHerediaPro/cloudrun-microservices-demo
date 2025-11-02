FROM maven:3.9-eclipse-temurin-17 AS build

WORKDIR /app

# Copia los archivos necesarios para el build de Maven
COPY pom.xml .
COPY src ./src

# Usa el comando 'mvn' global (ya que no tienes el wrapper 'mvnw')
# Esto debe funcionar ya que la imagen base es 'maven:...'
RUN mvn clean package -DskipTests

# ====================================================
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app

# **LÍNEA CRÍTICA:** Copia el JAR con el nombre esperado
# Confirma que el nombre exacto de tu JAR es 'user-service-1.0.0.jar'
COPY --from=build /app/target/user-service-1.0.0.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]