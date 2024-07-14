# CT519-Assign4
จาก Assign #3 ให้นักศึกษา สร้างการให้บริการด้วย Docker compose โดยให้มีการใช้งาน nginx server ที่ทำหน้าที่เป็น Reverse proxy เพื่อกระจายงาน ไปยัง web server ที่ทำงานบนทั้ง 2 คอนเทอเนอร์ โดยให้ เมื่อเข้าด้วย

   htttp://a.b.c.d/  ให้แสดงข้อมูลจาก  คอนเทนเนอร์ที่ 1  (ที่เปิดด้วยพอร์ต 80 ของ assign #3)
   htttp://a.b.c.d/about  ให้แสดงข้อมูลจาก  คอนเทนเนอร์ที่ 2  (ที่เปิดด้วยพอร์ต 8000 ของ assign #3)


ตัวอย่าง docker-compose.yaml ที่มี RP อยู่ด้านหน้า

services:
  reverse_proxy:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    networks:
      - frontend
      - backend

  ws1:
    image: nginx:alpine
    volumes:
      - ./ws1:/usr/share/nginx/html
    networks:
      - backend

  ws2:
    image: nginx:alpine
    volumes:
      - ./ws2:/usr/share/nginx/html
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge


ตัวอย่างไฟล์ nginx.conf ที่สามารถ
 http://ab.c.d/  LB ไปยัง web server ทั้ง 2 ตัว ด้วยวิธีการ Round Robin
 http://a.b.c.d/www1/  RP ไปยัง web server ตัวที่ 1 ที่ path  /www1
 htttp://a.b.c.d/www2/  RP ไปยัง web server ตัวที่ 2  ที่ path /www2

events {
    worker_connections 1024;
}

http {
    upstream backend {
        server ws1:80;
        server ws2:80;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        location /www1 {
            proxy_pass http://ws1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }

        location /www2 {
            proxy_pass http://ws2;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
