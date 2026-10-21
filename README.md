# Hello World: AWS Lambda & API Gateway en Java

Servicio REST que calcula el cuadrado de un entero, implementado como una función **AWS Lambda** en Java y expuesto con **AWS API Gateway**.

## Estructura

```
src/main/java/co/edu/escuelaing/services/MathServices.java   # función lambda
src/test/java/co/edu/escuelaing/services/MathServicesTest.java
pom.xml
```

```java
package co.edu.escuelaing.services;

public class MathServices {
    public static Integer square(Integer i) {
        return i * i;
    }
}
```

## Compilar

Requisitos: JDK 17 y Maven 3.

```bash
mvn package
```

Genera `target/math-services-1.0-SNAPSHOT.jar`, que es el archivo que se sube a Lambda.

> **Nota:** el tutorial original usa Java 8, pero AWS Lambda ya no permite crear funciones con ese runtime. Este proyecto compila para Java 17 y usa el runtime **Java 17**.

## 1. Crear la función Lambda

1. Consola AWS → **Lambda** → **Create function** → **Author from scratch**.
2. Nombre: `square`. Runtime: **Java 17**.
3. Permisos → **Use an existing role** → `LabRole` (AWS Academy).
4. En **Code** → **Upload from** → **.zip or .jar file** → subir `target/math-services-1.0-SNAPSHOT.jar`.
5. En **Runtime settings** → **Edit** → Handler:
   ```
   co.edu.escuelaing.services.MathServices::square
   ```
6. Pestaña **Test** → crear evento `testSquare` con el contenido `5` (solo el número, sin JSON) → **Test**. Resultado esperado: `25`.

## 2. Configurar API Gateway

1. Consola AWS → **API Gateway** → **REST API** → **Build** → **New API**, nombre `mathServices`, endpoint **Regional**.
2. Sobre el recurso `/` → **Create method** → **GET** → integración **Lambda function** → seleccionar `square`.
3. **Method request** → **URL query string parameters** → agregar `value` (marcar *Required*).
4. **Integration request** → **Edit**:
   - Desactivar *Lambda proxy integration*.
   - **URL query string parameters**: nombre `value`, mapeado desde `method.request.querystring.value`.
   - **Mapping templates** → Content type `application/json` → plantilla:
     ```
     $input.params("value")
     ```
5. Pestaña **Test** → query string `value=14` → resultado esperado `196`.
6. **Deploy API** → **New stage** → nombre `Beta`.
7. Invocar la URL del stage:
   ```
   https://<api-id>.execute-api.us-east-1.amazonaws.com/Beta?value=14
   ```
   Respuesta: `196`.

## 3. Limpieza

Al terminar, borrar la función Lambda `square` y el API `mathServices` para no generar costos.

## Evidencias

<!-- Agregar capturas en la carpeta images/ -->

| Paso | Captura |
|------|---------|
| Función Lambda creada | ![](images/lambda.png) |
| Test de Lambda con `5` → `25` | ![](images/lambda-test.png) |
| Método GET en API Gateway | ![](images/gateway-method.png) |
| Test en API Gateway con `value=14` | ![](images/gateway-test.png) |
| Invocación desde el navegador | ![](images/browser.png) |

## Autor

Sebastian Albarracin
