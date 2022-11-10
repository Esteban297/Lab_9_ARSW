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
   * Aumentar la capacidad de la máquina, nos permite disminuir el tiempo que toma calcular un número n en la serie de Fibonacci, por lo tanto el tiempo de respuesta será menor en la medida en que el hardware sea mejor, sin embargo, esto no garantiza que las peticiones siempre sean exitosas, lo cual requerirá otro tipo de soluciones.


9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?
   * No se genera un efecto negativo en el objetivo de la práctica, pues se hizo con el fín de mejorar el rendimiento del sistema a la hora de calcular la serie de fibonacci para un número aleatorio.

   Por otro lado, el escalamiento genera un costo adicional que es proporcional al tamaño que va aumentando la VM.


10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?
Lue
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
   ![image](https://user-images.githubusercontent.com/90571387/200722878-57d4f072-5e54-4573-95ba-b3c234b505c6.png)


2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

![](images/part2/part2-lb-bp-create.png)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![](images/part2/part2-lb-hp-create.png)
![image](https://user-images.githubusercontent.com/90571387/200724245-5acef945-a860-4fe7-b1ce-981eeca9d55f.png)


4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![](images/part2/part2-lb-lbr-create.png)
![image](https://user-images.githubusercontent.com/90571387/200725032-d8574532-9d4d-4aef-870a-9842724a0d11.png)



5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![](images/part2/part2-vn-create.png)
![image](https://user-images.githubusercontent.com/90571387/200727337-c5bf21b5-197c-4c7f-9710-bd0f7c5c897b.png)


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

![image](https://user-images.githubusercontent.com/90571387/200733333-8b12f12b-02e0-4513-8fac-c8abc0fb63fb.png)


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

## Documentación

### Tiempos de Respuesta

#### Timpos obtenidos realizando las pruebas con newman en cada máquina

    * VM1-r1

    - Tiempo: 2 minutos 27.8 segundos

![image](https://user-images.githubusercontent.com/90571387/200733723-df1d503b-7470-4f6c-96d2-1bef886fca4d.png)

    * VM1-r2

    - Tiempo: 2 minutos 28.6 segundos

![image](https://user-images.githubusercontent.com/90571387/200733748-5e3c174d-ec71-4fac-98e0-d672b68ddf5d.png)

    * VM2-r1

    - Tiempo: 2 minutos 28.4 segundos
![image](https://user-images.githubusercontent.com/90571387/200733767-14b0fd59-2de5-43c6-8236-9a8bf6406ac1.png)

    * VM2-r2

    - Tiempo: 2 minutos 23.6 segundos

![image](https://user-images.githubusercontent.com/90571387/200733798-14ea8c92-a2df-4d88-87a0-a06c3df196b9.png)

    * VM3-r1

    - Tiempo: 2 minutos 23.1 segundos

![image](https://user-images.githubusercontent.com/90571387/200733822-ee6c4f78-215c-48ff-961b-89421cb66e3a.png)

    * VM3-r2

    - Tiempo: 2 minutos 27.8 segundos

![image](https://user-images.githubusercontent.com/90571387/200733855-aec4b881-fa27-44f8-b73d-d78304ee65b2.png)

#### Timpos obtenidos realizando las pruebas con newman en cada máquina al tiempo

    * VM1-r1

    - Tiempo: 3 minutos 21.5 segundos

![image](https://user-images.githubusercontent.com/90571387/200733880-e5a7ea6d-92fd-4960-84b6-48f524c5e88f.png)

    * VM1-r2

    - Tiempo: 3 minutos 21.5 segundos

![image](https://user-images.githubusercontent.com/90571387/200733923-723caaad-9b29-4571-b316-9ee3cd774a9b.png)

    * VM2-r1

    - Tiempo: 3 minutos 15.9 segundos

![image](https://user-images.githubusercontent.com/90571387/200733959-bb950799-7a1d-45a4-9f4c-54a5fc8ee334.png)

    * VM2-r2

    - Tiempo: 3 minutos 19.3 segundos

![image](https://user-images.githubusercontent.com/90571387/200733989-d1b3c378-3d75-429a-a240-846f77ba8031.png)

    * VM3-r1

    - Tiempo: 2 minutos 43.7 segundos

![image](https://user-images.githubusercontent.com/90571387/200734018-352e3ec1-9436-4414-bdcc-a6f7f79b9486.png)

    * VM3-r2

    - Tiempo: 3 minutos 44.7 segundos

![image](https://user-images.githubusercontent.com/90571387/200734057-b2433ced-3fd2-4b25-8d83-b67742bbc81b.png)


### Peticiones Exitosas

#### Resultados exitosos realizando las pruebas con newman en cada máquina

![image](https://user-images.githubusercontent.com/90571387/200735027-4ff8c246-95b3-49d6-a53c-96a6b9407067.png)

#### Resultados exitosos realizando las pruebas con newman en cada máquina al tiempo

![image](https://user-images.githubusercontent.com/90571387/200734967-2027376c-889c-4382-a8fd-31266c4db1a5.png)

- En el escalamiento vertical no se obtuvo ninguna respuesta exitosa

### Costos

![image](https://user-images.githubusercontent.com/90571387/200735278-23947377-0b1c-4d68-b675-b64aad76fdd7.png)


### Resultados VM4

    * VM4-r1

    - Tiempo: 2 minutos 42.8 segundos

![image](https://user-images.githubusercontent.com/90571387/200734235-2030deee-d1cf-4772-9f0a-fb3f68f807b7.png)

    * VM4-r2

    - Tiempo: 2 minutos 53.1 segundos

![image](https://user-images.githubusercontent.com/90571387/200734257-f52fd422-c3e9-47b3-b8e7-c48f732ff17b.png)

    * VM4-r3

    - Tiempo: 2 minutos 57.6 segundos

![image](https://user-images.githubusercontent.com/90571387/200734292-d1ff2e3c-09d4-4903-924c-044e40661abb.png)

    * VM4-r4

    - Tiempo: 3 minutos 0.1 segundos

![image](https://user-images.githubusercontent.com/90571387/200734318-740d81d7-13a1-43df-a8b8-8d6e1f496bcf.png)


#### Comportamiento de CPU

    * VM1

![image](https://user-images.githubusercontent.com/90571387/200734354-1e72783d-6264-4d49-ad24-706b033d0acd.png)

    * VM2

![image](https://user-images.githubusercontent.com/90571387/200734370-e7f120bf-bd56-46d2-8842-61547f61ccba.png)

    * VM3

![image](https://user-images.githubusercontent.com/90571387/200734390-3be455fd-f6a1-4f30-93c6-c76d98d4f809.png)

    * VM4

![image](https://user-images.githubusercontent.com/90571387/200734407-6edf68da-661f-49b8-933c-238d397c5f50.png)

- La tasa de éxito aumentó con el estilo de escalabilidad horizontal, ya que además de aumentar el número de servidores que atienden las peticiones, también se creó un balanceador de carga para controlar el tráfico en la aplicación; de modo que estas solicitudes se reparten entre estas máquinas equilibradamente y se aumenta así el número de usuarios/clientes concurrentes que pueden ser atendidos.

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?
   * Actualmente existen dos tipos de balanceo de cargas, por un lado tenemos el balanceo de carga público, el cual se encarga de proveer conexiones de salida para las maquinas virtuales que conforman el backend, esto es posible mediante la traducción de direcciones IP privadas a públicas, y por otro lado tenemos el balanceador de cargas interno o privado, el cual se encarga de balacear las cargas dentro de la red interna de nuestra red virtual.
   * SKU en microsoft azure, son los niveles de recursos que están disponibles para los clientes dependiendo de la tarea que se va a realizar, estos se clasifican en cuatro categorias las cuales son: Free, Basic, Standard y Storage Optimized. Cada una de ellas, determina el monto que el cliente debe pagar a la hora de usar el servicio.
   * Cada categoría brinda un nivel de capacidad diferente y se eligen dependiendo del tamaño del proyecto que se quiere desplegar.
   * El balanceador de carga requiere una ip pública, porque es ahí donde llegan todas las peticiones que se realizan, de ahí se distribuye a las demás máquinas para que realicen en cálculo y así retornar la respuesta.
* ¿Cuál es el propósito del *Backend Pool*?
   * Es una agrupación lógica de sus instancias de aplicaciones en todo el mundo que reciben el mismo tráfico y responden con el comportamiento esperado
   * Un backend pool define cómo se deben evaluar los diferentes backends a través de sondas de estado. También define cómo se produce el equilibrio de carga entre ellos.
* ¿Cuál es el propósito del *Health Probe*?
   * El Health Probe, se encarga de determinar el estado de cada una de las máquinas que conforman el backend pool mediante peticiones de prueba, esto con el fin de ajustar el balanceo de carga entre las diferentes instancias.
* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.
   * El propósito de la Load balancing rule es definir como se distribuye el tráfico a las máquinas virtuales con el fín de definir el puerto por la cual a escuchar al front end, esto es importante, ya que el frontend puede enviar trafico de red a las máquinas virtuales que están en el backend con balanceo de carga y evitar errores en las respuestas a las peticiones.
* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?
   * Una Virtual Network es una representación de una red propia en la nube. Es un aislamiento lógico de la nube de Azure dedicada a la suscripción del usuario. Puede utilizar VNets para aprovisionar y gestionar redes virtuales privadas (VPNs) en Azure y, opcionalmente, enlazar las VNets con otras VNets en Azure, o con una infraestructura de TI local.
   * Una Subnet es un rango de direcciones lógicas. Es una buena estrategia si se tiene una red de gran tamaño ya que al dividirla en subredes puede reducir el tamaño de dominios de broadcast y hacerla más fácil de administrar.
   * Address space: al crear una red virtual, debe especificar un espacio de direcciones IP privadas personalizadas utilizando direcciones públicas y privadas (RFC 1918). Azure asigna recursos en una red virtual a una dirección IP privada desde el espacio de direcciones que asigne. Por ejemplo, si implementa una VM en una red virtual con espacio de direcciones, 10.0.0.0/16, a la VM se le asignará una IP privada como 10.0.0.4.
   * Address range es el rango de direcciones ip disponibles para poder configurar la red virtual, por defecto azure asigna una mascara de red de /28 o /16 y se puede realizar subredes con diferentes rangos, dentro del rango maximo
* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?
   * Availability Zone es una oferta que se realiza para tener alta disponibilidad ya que potege sus aplicaciones y datos a fallas que puedan ocurrir. las zonas de disponibilidad son ubicaicones ficticias dentro de una region en azure.
* ¿Cuál es el propósito del *Network Security Group*?
   * La Network Security Group se ecarga de restringir el tráfico que llega a las máquinas virtuales, esto con el fin de fortalecer el nivel de seguridad y regular las peticiones que llegan al sistema
* Informe de newman 1 (Punto 2)
   * Como lo pudimos ver en la documentación, el sistema se demoró en responder las peticiones en promedio 2 min 53 seg, de las cuales, solo se presentaban entre 1 y 3 fallas. En comparación con el sistema de 3 máquinas, este se demoró menos tiempo en retornar los resultados al cliente.
* Presente el Diagrama de Despliegue de la solución.
   * ![image](https://user-images.githubusercontent.com/90571387/200982300-e683d2a4-5dee-4a21-a401-c06f1f5b25a2.png)




