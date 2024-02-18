# Use official Nginx image as base
FROM nginx:alpine

# Copy the HTML, CSS, and JavaScript files to the appropriate directory
COPY index.html /usr/share/nginx/html/
COPY style.css /usr/share/nginx/html/
COPY script.js /usr/share/nginx/html/

# Expose port 80 to make the app accessible
EXPOSE 80

# Define the command to start Nginx when the container starts
CMD ["nginx", "-g", "daemon off;"]
