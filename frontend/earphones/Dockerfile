# Use lightweight NGINX image
FROM nginx:alpine

# Copy all frontend files into NGINX html folder
COPY . /usr/share/nginx/html

# Expose port 80
EXPOSE 80

# Start NGINX
CMD ["nginx", "-g", "daemon off;"]
