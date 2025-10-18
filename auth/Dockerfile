# Dùng Node 20 làm môi trường chạy
FROM node:20

# Tạo thư mục làm việc trong container
WORKDIR /app

# Sao chép file package vào trước để tối ưu cache
COPY package*.json ./

# Cài các dependencies
RUN npm install

# Sao chép toàn bộ source code vào container
COPY . .

# Expose cổng chạy service (3000)
EXPOSE 3000

# Lệnh khởi động container
CMD ["npm", "start"]