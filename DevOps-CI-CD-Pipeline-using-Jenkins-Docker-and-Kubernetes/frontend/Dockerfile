# Use official NGINX image
FROM nginx:alpine

# Copy the static html file to nginx's html directory
COPY index.html /usr/share/nginx/html/index.html

# Expose port 80
EXPOSE 80

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
