# Cloud Engineer Sprint 2

## Crea un ambiente de Cloud9 con las siguientes características

- Name: cs2-paas
- Type: t2.micro
- Platform: Amazon Linux 2

## 1. Clona el repositorio

```
git clone https://github.com/lemmhubaws-cloud-engineer-sprint-2/tree/main
```

1.  Descomprime el repositorio

```
unzip aws-cloud-engineer-sprint-2/CS_2.zip -d CS_
```

## 2. Crear las bases de datos

1. Ejecuta el siguiente comando

```
aws rds create-db-instance \
    --db-instance-identifier paas-db \
    --db-instance-class db.t3.micro \
    --engine mysql \
    --master-username admin \
    --master-user-password bootcampinstitute \
    --allocated-storage 20
```

2. Ejecuta el siguiente comando para obtener el id del security group

```
aws rds describe-db-instances | grep VpcSecurityGroupId
```

3. Modifica los inbound access del security group

```
aws ec2 authorize-security-group-ingress --group-id {security-group-obtenido} --protocol tcp --port 3306 --cidr 0.0.0.0/0
```

4. Obtén el Endpoint creado

```
aws rds describe-db-instances | grep Address
```

5. Prueba la conexión a la base de datos
   - Nota que las credenciales son:
     - user= admin
     - password = bootcampinstitute

```
 mysql -u admin -p -h {ingresa el endpoint obtenido}

 e.j.
  mysql -u admin -p -h paas-db.cnyxqkx1bbds.us-east-1.rds.amazonaws.com
```

![Image 1](/mdpic/1.PNG)


6. Crea una base de datos y tabla para almacenar la información

```
create Database pizzas;

CREATE TABLE `pizzas`.`catalogo` (
  `id` INT NOT NULL AUTO_INCREMENT,
  `descripcion` VARCHAR(100) NULL,
  `ingredientes` VARCHAR(100) NULL,
  `precio` FLOAT NULL,
  `imagen` VARCHAR(1000) NULL,
  PRIMARY KEY (`id`));
```

7. Ingresa los datos del sitio dinámico

```
INSERT INTO `pizzas`.`catalogo` (`descripcion`, `precio`, `ingredientes`,`imagen`) VALUES ('Hongos', '200', 'Zetas, Cheddar, Jitomate, Carnes Frías','https://ce-cloudsprint-2.s3.amazonaws.com/images/hongos.png');

  INSERT INTO `pizzas`.`catalogo` (`descripcion`, `precio`,`ingredientes`, `imagen`) VALUES ('Mexicana', '250', 'Chorizo, Cheddar, Aguacate', 'https://ce-cloudsprint-2.s3.amazonaws.com/images/mexicana.png');

  INSERT INTO `pizzas`.`catalogo` (`descripcion`, `precio`, `ingredientes`,`imagen`) VALUES ('Pepperoni', '250', 'Pepperoni, Cheddar, Jitomate','https://ce-cloudsprint-2.s3.amazonaws.com/images/pepperoni.png');

  INSERT INTO `pizzas`.`catalogo` (`descripcion`, `precio`,`ingredientes`, `imagen`) VALUES ('Salami', '200','Salami, Cheddar, Jitomate', 'https://ce-cloudsprint-2.s3.amazonaws.com/images/salami.png');

  INSERT INTO `pizzas`.`catalogo` (`descripcion`, `precio`, `ingredientes`,`imagen`) VALUES ('Suprema', '280','Pepperoni, Salami, Zetas, Cheddar, Jitomate', 'https://ce-cloudsprint-2.s3.amazonaws.com/images/suprema.png');
```

8. Modifica el archivo index.js para tomar los valores de la base de datos:

- host: “{endpoint-de-rds}”
- User: {usuario-de-base-de-datos}
- Password:{el-password-del-usuario}
- Database: pizzas

## 3. Probar el sitio de manera interna

1. Posiciónate dentro de la terminal en el archivo del repositorio CS_2

```
cd CS_2/
```

2. Instala los paquetes de node y ejecuta

```
npm install
npm start
```

![Image 2](/mdpic/2.PNG)

3. Puedes previsualizar la aplicación haciendo click en Preview

## 4. Integrar con Route53

1. Obtén el id del Grupos de seguridad para permitir el acceso al puerto 8080

```
aws ec2 describe-security-groups  --filters Name=group-name,Values=*cs2* | grep GroupId
```

-- Nota, puedes obtener el nombre completo del security group crado con el siguiente comando (deberás sustituirlo por cs2 si el sg toma otro nombre)

```
 curl -w "\n" http://169.254.169.254/latest/meta-data/security-groups
```

2.  Modifica los inbound access del security group

```
aws ec2 authorize-security-group-ingress --group-id {security-group-obtenido} --protocol tcp --port 8080 --cidr 0.0.0.0/0
```

3. Obtén la DNS de la instancia

```
curl -w "\n" http://169.254.169.254/latest/meta-data/public-hostname
```

4. Ejecuta nuevamente los paquetes de node y revisa el sitio desde la página web, deberás ingresar el sufijo del puerto :8080

e.j. http://ec2-paas-xxx-xxx-xxx-xxx.compute-1.amazonaws.com:8080/

5. Obtén la Ip pública de la instancia ec2 y el id del host registrado en Route53

```
#ip públiccca
curl -w "\n" http://169.254.169.254/latest/meta-data/public-ipv4

#Id route53
aws route53 list-hosted-zones
```

6. Modifica el archivo **route53.json**

   - "Name": Nombre del dominio registrado en freenom
   - ResourseRecords: Ingresa la Ip pública de la instancia

7. Ejecuta el registro de Route53

```
aws route53 change-resource-record-sets --hosted-zone-id {ingresa id  hosted zone} --change-batch file://route53.json
```

8. Verifica que el dominio de freenom haya registrado los cambios y  despliegue el sitio dinámico
