### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/es-es/free/students/). Al hacerlo usted contará con $100 USD para gastar durante 12 meses.

### Parte 0 - Entendiendo el escenario de calidad

Adjunto a este laboratorio usted podrá encontrar una aplicación totalmente desarrollada que tiene como objetivo calcular el enésimo valor de la secuencia de Fibonnaci.

**Escalabilidad**
Cuando un conjunto de usuarios consulta un enésimo número (superior a 1000000) de la secuencia de Fibonacci de forma concurrente y el sistema se encuentra bajo condiciones normales de operación, todas las peticiones deben ser respondidas y el consumo de CPU del sistema no puede superar el 70%.

### Parte 1 - Escalabilidad vertical

1. Diríjase a el [Portal de Azure](https://portal.azure.com/) y a continuación cree una maquina virtual con las características básicas descritas en la imágen 1 y que corresponden a las siguientes:
    * Resource Group = SCALABILITY_LAB
    * Virtual machine name = VERTICAL-SCALABILITY
    * Image = Ubuntu Server 
    * Size = Standard B1ls
    * Username = scalability_lab
    * SSH publi key = Su llave ssh publica

![Imágen 1](images/part1/part1-vm-basic-config.png)

![image](https://user-images.githubusercontent.com/90571387/200700370-6d8a99c9-d34f-4e1b-8ddd-b6abfd01e202.png)

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM (Revise la sección "Connect" de la virtual machine creada para tener una guía más detallada).

    `ssh scalability_lab@xxx.xxx.xxx.xxx`
    
   ![image](https://user-images.githubusercontent.com/90571387/200700461-441da879-16cd-42e8-a258-2a8c9f97ef8d.png)


3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).
   ![image](https://user-images.githubusercontent.com/90571387/200700489-5f92c27b-f96c-4bf3-a479-e8cb654552e0.png)
4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`
    
 
    ![image](https://user-images.githubusercontent.com/90571387/200700566-b6f635d3-122e-46ff-b7f4-7eaf714c16ec.png)



5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    ` node FibonacciApp.js`
   ![image](https://user-images.githubusercontent.com/90571387/200700712-7da6b51a-63d0-4abd-acdf-7c27785a4201.png)

   
6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

![](images/part1/part1-vm-3000InboudRule.png)
  

![image](https://user-images.githubusercontent.com/90571387/200700832-66bcb745-0ace-48ce-a248-39df994962d1.png)


7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:

    * 1000000
   
      - Tiempo: 2,9 m
      ![image](https://user-images.githubusercontent.com/90571387/200700932-5e6443ad-08a5-4357-9225-ff93bc0b3a7f.png)

    * 1010000
    
      - Tiempo 3,4 m
      ![image](https://user-images.githubusercontent.com/90571387/200704757-766d6068-fce6-4f2f-92f7-9fb4c202d397.png)

    * 1020000
    
      - Tiempo 3,1 m
      ![image](https://user-images.githubusercontent.com/90571387/200704813-45febbea-39ac-4df1-83be-6b0f4b511607.png)

    * 1030000
    
      - Tiempo 3,2 m
      ![image](https://user-images.githubusercontent.com/90571387/200704866-5044dfae-73aa-407e-8903-7d6c6929c1b5.png)

    * 1040000
      
      - Tiempo 3,2 m
      ![image](https://user-images.githubusercontent.com/90571387/200704906-91da77ae-5494-42ee-b5ab-89192b0c7e9a.png)

    * 1050000
    
      - Tiempo 3,3 m
      ![image](https://user-images.githubusercontent.com/90571387/200704941-8ad81ba0-5573-45b8-b083-ea652fedf86b.png)

    * 1060000
    
      - Tiempo 3,4 m
      ![image](https://user-images.githubusercontent.com/90571387/200704973-c8a69249-6521-494b-bdc7-cee32f3ccaf2.png)

    * 1070000
    
      - Tiempo 3,5 m
      ![image](https://user-images.githubusercontent.com/90571387/200705014-09fc97f3-1a26-4f87-8d36-ef94f532ed35.png)

    * 1080000
     
      - Tiempo 3,4 m
      ![image](https://user-images.githubusercontent.com/90571387/200705037-12001311-d2de-425d-a79e-0a24a67832b1.png)

    * 1090000   
    
      - Tiempo 9,5 m
      ![image](https://user-images.githubusercontent.com/90571387/200705087-5be7014e-f8bc-4d7f-9913-139bb95ab225.png)


8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

![Imágen 2](images/part1/part1-vm-cpu.png)

![image](https://user-images.githubusercontent.com/90571387/200705679-56be26e8-fb59-4b40-bfc5-750998e12769.png)


9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
   
   ![image](https://user-images.githubusercontent.com/90571387/200705796-ee59a3b8-5b6e-4589-bf1a-979233ae0855.png)

    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```

![image](https://user-images.githubusercontent.com/90571387/200712551-f6a6ff6b-daea-4e46-9ba3-11e4ff5d4181.png)
![image](https://user-images.githubusercontent.com/90571387/200712655-7ceaf6a5-6bf9-43ed-a674-df764082424d.png)
![image](https://user-images.githubusercontent.com/90571387/200712683-6bbc4077-9106-4f12-9515-842818bfadd1.png)
![image](https://user-images.githubusercontent.com/90571387/200712707-132b0cec-6d9b-44c6-b080-4688329f123b.png)


10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.

     ##Con tamaño B2ms
      
      * 1000000

         - Tiempo: 38,13 s
         ![image](https://user-images.githubusercontent.com/90571387/200714369-904806bb-ccac-45d0-bf63-bda0fab615ce.png)


      * 1010000

         - Tiempo 27,16 s
         ![image](https://user-images.githubusercontent.com/90571387/200714511-5c20376f-9d25-4c77-ad75-9c2bcd5ac3e7.png)

       * 1020000

         - Tiempo 27,64 s
         ![image](https://user-images.githubusercontent.com/90571387/200714633-91b471f3-5c2c-42b4-bcc9-a0c92000cc71.png)

       * 1030000

         - Tiempo 28,25 s
         ![image](https://user-images.githubusercontent.com/90571387/200714769-77e07e68-e9cc-481e-adbe-a047aae04fb9.png)

       * 1040000

         - Tiempo 28,84 s
         ![image](https://user-images.githubusercontent.com/90571387/200715327-12639ed3-d972-4fd1-b9ec-f6770802991a.png)

       * 1050000

         - Tiempo 29,31 s
         ![image](https://user-images.githubusercontent.com/90571387/200715425-e6db87fc-51b2-4ed5-85d8-c6c4ed3b159d.png)

       * 1060000

         - Tiempo 29,87 s
         ![image](https://user-images.githubusercontent.com/90571387/200715617-c65dfd9f-97a1-49eb-b4c2-1f36befdc7db.png)

       * 1070000

         - Tiempo 30,41 s
         ![image](https://user-images.githubusercontent.com/90571387/200715744-485510e0-cf4d-4598-8b3a-c7c9c1e69678.png)

       * 1080000

         - Tiempo 30,88 s
         ![image](https://user-images.githubusercontent.com/90571387/200715861-d9a144da-790d-47f9-b9fc-9c24083e7408.png)

       * 1090000   

         - Tiempo 31,55 s
         ![image](https://user-images.githubusercontent.com/90571387/200715991-9dc06e7d-7bf5-48c7-a3aa-ea662b900533.png)


  * Consumo de CPU
  ![image](https://user-images.githubusercontent.com/90571387/200716346-2df53bbd-2418-4a2c-8981-779188931caa.png)

  * Postman
  
  ![image](https://user-images.githubusercontent.com/90571387/200716974-b8ff982f-1df6-41d0-ad7b-088564f546fe.png)
  ![image](https://user-images.githubusercontent.com/90571387/200717049-5d5187bb-974c-4196-825f-a179cdb0b62e.png)
  ![image](https://user-images.githubusercontent.com/90571387/200717084-241edc9c-17d8-4f15-9e09-06862b9348d7.png)
  ![image](https://user-images.githubusercontent.com/90571387/200717117-80a28a76-2d22-4a97-bf7b-80b06f7311a1.png)


12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.
   
   RTA: Se puede Considerar que si cumple un requerimiento funcional, debido a que se ve una mejora de tiempo muy grande pasando de 10 minutos a 30 segundos, de igual manera hubo disminución en los errores de scripts en postman de 5 a 4.
   
13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?
   * En este caso se crean cuatro recursos que son: Virtual Machine, Resource Group, Public and Private IP address.
2. ¿Brevemente describa para qué sirve cada recurso?
   * Virtual Machine: Es un software que simula a un computador real.
   * Resource Group: Es un contenedor que contiene recursos relacionados para una solución de Azure. El grupo de recursos incluye aquellos recursos que desea administrar como grupo.
   * Public IP address: Las direcciones IP públicas permiten que los recursos de Internet se comuniquen de forma entrante a los recursos de Azure. Las direcciones IP públicas permiten que los recursos de Azure se comuniquen con Internet y con los servicios públicos de Azure.
   * IP privada: Permiten la comunicación entre recursos en Azure tales como:
      * Virtual machine network interfaces
      * Internal load balancers (ILBs)
      * Application gateways
      * Virtual network.
      * Red local a través de una pasarela VPN o un circuito ExpressRoute.
3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?
   * Si se cierra la conexión ssh también se terminaran los procesos que se esten ejecutando en esa conexión, por eso es necesario ejecutarlo con forever. Porque por defecto solo esta abierto el puerto 22 por lo que las conexiones desde el resto de puertos no son aceptadas
4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.
   * El alto consumo de tiempo de respuesta, puede deberse a que actualmente estamos trabajando con una máquina virtual que posee unas características de gama baja, es decir, posee un hardware insuficiente que no le permite realizar tareas complejar como calcular la serie de Fibonacci para un número grande.


   ![image](https://user-images.githubusercontent.com/90571387/200719211-238f0b3b-5d37-4864-9ba7-397a6193f51e.png)

5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.
   * La función consume esa cantidad de cpu, porque debe almacenar números en cada iteración del ciclo for, estos numeros van aumentando en tamaño a una tasa de incremento constante, lo cual implica que la cpu debe invertir más recursos a medida que va aumentando de tamaño el número.
   ![image](https://user-images.githubusercontent.com/90571387/200719509-55caaa3a-e307-4044-b178-8d3677682267.png)

6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.
    ![image](https://user-images.githubusercontent.com/90571387/200719610-1d4db15f-a505-458e-9568-3cda0acf7be0.png)
    * Si hubo fallos documentelos y explique.
      * Los fallos pueden causarse por timeouts, debido a que el tiempo de espera, puede ser un poco tardado.
7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?
   * B1ls es la máquina más basica que tien azure, la cual solo cuenta con 1 vcpu y 0.5 Gb de memoria RAM, por otro lado B2ms, posee mayor capacidad, contando con 2vcpu y 8 Gb de menoria RAM, que le permite manejar de una manera más adecuada los recursos, con la capacidad de usar una mayor cantidad de entradas y obtener una mejor disponibilidad en comparación a la B1ls.
8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?
   Aumentar la capacidad de la máquina, nos permite disminuir el tiempo que toma calcular un número n en la serie de Fibonacci, por lo tanto el tiempo de respuesta será menor en la medida en que el hardware sea mejor, sin embargo, esto no garantiza que las peticiones siempre sean exitosas, lo cual requerirá otro tipo de soluciones.


9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?
   No se genera un efecto negativo en el objetivo de la práctica, pues se hizo con el fín de mejorar el rendimiento del sistema a la hora de calcular la serie de fibonacci para un número aleatorio.

   Por otro lado, el escalamiento genera un costo adicional que es proporcional al tamaño que va aumentando la VM.


10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?
11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?
   ![image](https://user-images.githubusercontent.com/90571387/200720790-ec8cd9d4-4728-4197-b3de-0a34c1ccd0cb.png)
   ![image](https://user-images.githubusercontent.com/90571387/200720854-95e1590b-f419-4a48-924a-8d1f42f4b814.png)
   ![image](https://user-images.githubusercontent.com/90571387/200720927-0c3c4d2f-9e33-40b5-91ab-9ec976a25a0b.png)
   * Como podemos ver, si hubo una mejoria, ya que de los scripts ejecutados, ninguno fue fallido.
  
### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

![](images/part2/part2-lb-create.png)

2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

![](images/part2/part2-lb-bp-create.png)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![](images/part2/part2-lb-hp-create.png)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![](images/part2/part2-lb-lbr-create.png)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![](images/part2/part2-vn-create.png)

#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.

1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.

![](images/part2/part2-vm-create1.png)

2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.

![](images/part2/part2-vm-create2.png)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

![](images/part2/part2-vm-create3.png)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.

![](images/part2/part2-vm-create4.png)

5. Finalmente debemos instalar la aplicación de Fibonacci en la VM. para ello puede ejecutar el conjunto de los siguientes comandos, cambiando el nombre de la VM por el correcto

```
git clone https://github.com/daprieto1/ARSW_LOAD-BALANCING_AZURE.git

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
source /home/vm1/.bashrc
nvm install node

cd ARSW_LOAD-BALANCING_AZURE/FibonacciApp
npm install

npm install forever -g
forever start FibonacciApp.js
```

Realice este proceso para las 3 VMs, por ahora lo haremos a mano una por una, sin embargo es importante que usted sepa que existen herramientas para aumatizar este proceso, entre ellas encontramos Azure Resource Manager, OsDisk Images, Terraform con Vagrant y Paker, Puppet, Ansible entre otras.

#### Probar el resultado final de nuestra infraestructura

1. Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:

```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1
```

2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?
* ¿Cuál es el propósito del *Backend Pool*?
* ¿Cuál es el propósito del *Health Probe*?
* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.
* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?
* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?
* ¿Cuál es el propósito del *Network Security Group*?
* Informe de newman 1 (Punto 2)
* Presente el Diagrama de Despliegue de la solución.




