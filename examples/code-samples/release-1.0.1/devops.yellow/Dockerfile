# Build command
# Adjust docker registry path (npantelic/...) into (your-docker-username/...)
#
# 1. [Linux/Mac] : ./gradlew clean bootJar 
#    [Windows]   : .\gradlew clean bootJar
# 2. docker build --tag devops-yellow:1.0.1 . 
# 3. docker login (optional)
# 4. docker tag devops-yellow:1.0.1 npantelic/devops-yellow:1.0.1
# 5. docker push npantelic/devops-yellow:1.0.1

# .\gradlew clean bootJar && docker build --tag devops-yellow:1.0.1 . && docker tag devops-yellow:1.0.1 npantelic/devops-yellow:1.0.1 && docker push npantelic/devops-yellow:1.0.1

# Start with a base image containing Java runtime
FROM amazoncorretto:17

# The application's jar file
ARG JAR_FILE=build/libs/devops-yellow.jar

# Add the application's jar to the container
ADD ${JAR_FILE} devops-yellow.jar

# Run the jar file 
ENTRYPOINT ["java", "-jar", "devops-yellow.jar"]