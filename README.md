# Setup
## Init
```bash
export $PROJECT
docker-compose up -d --build
docker exec -it ${PROJECT}_php_1 php init
docker exec -it ${PROJECT}_php_1 composer install
```
## Migrations

```bash
docker exec -it ${PROJECT}_php_1 php yii migrate
docker exec -it ${PROJECT}_php_1 php yii rbac/migrate
docker exec -it ${PROJECT}_php_1 php yii user/create admin@project.dev admin password
docker exec -it ${PROJECT}_php_1 php yii user/role/assign admin admin@project.dev
```
