# Build command
# Adjust docker registry path (npantelic/...) into (your-docker-username/...)
#
# 1. [Linux/Mac] : ./gradlew clean bootJar 
#    [Windows]   : .\gradlew clean bootJar
# 2. docker build --tag devops-white:1.0.0 . 
# 3. docker login (optional)
# 4. docker tag devops-white:1.0.0 npantelic/devops-white:1.0.0
# 5. docker push npantelic/devops-white:1.0.0

# .\gradlew clean bootJar && docker build --tag devops-white:1.0.0 . && docker tag devops-white:1.0.0 npantelic/devops-white:1.0.0 && docker push npantelic/devops-white:1.0.0

# Start with a base image containing Java runtime
FROM amazoncorretto:17

# The application's jar file
ARG JAR_FILE=build/libs/devops-white.jar

# Add the application's jar to the container
ADD ${JAR_FILE} devops-white.jar

# Run the jar file 
ENTRYPOINT ["java", "-jar", "devops-white.jar"]