# Taller glusterfs

## Definiciones

[GlusterFS](https://www.gluster.org/) es un sistema de archivos en red escalable y distribuído, definido para ser utilizado en el espacio de usuario, es decir, sin utilizar el espacio de almacenamiento crítico del sistema, y de esta forma no compromete el rendimiento. 

* Gluster es una forma sencilla de aprovisionar su propio backend de almacenamiento NAS utilizando casi cualquier hardware que elija.
* La recuperación de errores (failover) se hace de forma automática, de modo que si un servidor se cae, no se pierde el acceso a los datos. Cuando se recupera el servidor que ha fallado no hay que hacer nada para recuperar los datos, excepto esperar. Mientras tanto, la copia más actual de los datos se sigue sirviendo desde los nodos que estaban operativos.
* Se puede acceder a los datos de Gluster desde los clientes tradicionales NFS, SMB/CIFS o usando el cliente nativo.

Ventajas:

* **Simplicidad**: Es fácil de utilizar y al ser ejecutado en el espacio de usuario es independiente del núcleo.
* **Elasticidad**: Se adapta al crecimiento y reduce el tamaño de los datos.
* **Escalabilidad**: Tiene disponibilidad de Petabytes y más.
* **Velocidad**: Elimina los metadatos y mejora el rendimiento considerablemente unificando los datos y objetos.

Conceptos:

* El **"trusted pool"** (pool de confianza) es el término utilizado para definir un cluster de nodos en Gluster.
* **brick** (ladrillo): será el directorio (donde hemos montado un dispositivo de bloque) que compartirá cada nodo del clúster. 
* **Volumen**: Un volumen es una colección lógica de Bricks.

## Tipos de volúmenes

* **Volumen distribuido**: Este es el tipo de volumen por defecto en GlusterFS, si no se especifica un tipo de volumen concreto. Los archivos se distribuyen en varios bricks en el volumen, de forma que el archivo 1 sólo podrá almacenarse en el brick 1 o en el brick 2, pero no en ambos, por lo que no habrá redundancia de datos. Este tipo de volumen distribuido hace que sea más fácil y barato escalar el tamaño del volumen. No obstante, este tipo de volumen, al no proporcionar redundancia, puede sufrir la pérdida de los datos en caso de que uno de los dos bricks falle, por lo que es necesario realizar un backup de los archivos con una aplicación externa a GlusterFS.

![](https://cloud.githubusercontent.com/assets/10970993/7412364/ac0a300c-ef5f-11e4-8599-e7d06de1165c.png)

* **Volumen replicado**: Con este tipo de volumen eliminamos el problema ante la pérdida de datos que se experimenta con el volumen distribuido. En el volumen replicado se mantiene una copia exacta de los datos en cada uno de los bricks. El número de réplicas se configura por el usuario al crear el volumen, si queremos dos réplicas necesitaremos al menos dos bricks, si queremos tres réplicas necesitaremos tres bricks, y así sucesivamente. Si un brick está dañado, todavía podremos acceder a los datos mediante otro brick. Este tipo de volumen se utiliza para obtener fiabilidad y redundancia de datos.

![](https://cloud.githubusercontent.com/assets/10970993/7412379/d75272a6-ef5f-11e4-869a-c355e8505747.png)

* **Volumen distribuido replicado**: En este tipo de volumen los datos se distribuyen en conjuntos duplicados de bricks. El número de bricks debe ser un múltiplo del número de réplicas. También es importante el orden en que especifiquemos los bricks porque los bricks adyacentes serán réplicas entre ellos. Este tipo de volumen se utiliza cuando se necesita una alta disponibilidad de los datos debido a su redundancia a la escalabilidad del tamaño de almacenamiento. Si tenemos ocho bricks y configuramos una réplica de dos, los dos primeros bricks serán réplicas el uno del otro, y luego los dos siguientes, y así sucesivamente. Esta configuración se denomina 4×2. Si tenemos ocho bricks y configuramos una réplica de cuatro, los cuatro primeros bricks serán réplicas entre ellos y se denominará 2×4.

![](https://cloud.githubusercontent.com/assets/10970993/7412402/23a17eae-ef60-11e4-8813-a40a2384c5c2.png)

* **Volumen seccionado (Striped)**: Imagina un archivo de gran tamaño que se almacena en un brick al que se accede con frecuencia desde muchos clientes al mismo tiempo. Esto provocaría demasiada carga en un solo brick y reduciría considerablemente el rendimiento. En un volumen seccionado los archivos se almacenan en diferentes bricks después de haberse dividido en secciones, de forma que un archivo grande se divide en diferentes secciones y cada sección se almacena en un brick. De este modo se distribuye la carga y el archivo puede ser recuperado más rapidamente, pero en este tipo de volumen perderemos la redundancia de los datos.

![](https://cloud.githubusercontent.com/assets/10970993/7412387/f411fa56-ef5f-11e4-8e78-a0896a47625a.png)

* **Volumen seccionado distribuido**: Este volumen es similar al volumen seccionado, excepto que las secciones en este caso pueden ser distribuídas en más cantidad de bricks.

![](https://cloud.githubusercontent.com/assets/10970993/7412394/0ce267d2-ef60-11e4-9959-43465a2a25f7.png)

## Preparación del entorno

Los tres nodos se van a conocer por nombre, por lo que vamos a añadir a `/etc/hosts`:

```
10.1.1.101 nodo1
10.1.1.102 nodo2
10.1.1.103 nodo3
```

## Instalación

En los tres nodos:

```
apt update
apt install glusterfs-server
systemctl enable glusterd.service
systemctl start glusterd.service
```

## Formateo del dispositivo de bloque

En los tres nodos, formateamos y montamos el disco adicional:

```
mkfs.xfs /dev/vdb
mkdir -p /data/brick1
echo '/dev/vdb /data/brick1 xfs defaults 1 2' >> /etc/fstab
mount -a && mount
```

En cada nodo hemos creado un *brick* (ladrillo) que será el directorio (donde hemos montado un dispositivo de bloque) que compartirá cada nodo del clúster. 

## Configurar el pool de confianza

Elegimos el `nodo1` como nodo primario, y creamos el "trusted pool":

```
gluster peer probe nodo2
gluster peer probe nodo3
```

Ya tenemos añadidos los tres nodos al "trusted pool". Si ahora intentamos conectar del `nodo2` al `nodo1` vemos que ya está establecido la unión. Desde el `nodo2` ejecutamos:

```
root@nodo2:~# gluster peer probe nodo1
peer probe: Host nodo1 port 24007 already in peer list
```

Ahora podemos comprobar el estado del clúster, desde el `nodo1`:

```
root@nodo1:~# gluster peer status
Number of Peers: 2

Hostname: nodo2
Uuid: 2f8e94ab-e5ef-4af5-8fe1-117c56028ee9
State: Peer in Cluster (Connected)

Hostname: nodo3
Uuid: 38028702-90cd-4af6-8abf-f8d93bfa9d5a
State: Peer in Cluster (Connected)
```

O desde el `nodo2`:

```
root@nodo2:~# gluster peer status
Number of Peers: 2

Hostname: nodo1
Uuid: 6784c07c-90f5-4850-9c18-82108c216535
State: Peer in Cluster (Connected)

Hostname: nodo3
Uuid: 38028702-90cd-4af6-8abf-f8d93bfa9d5a
State: Peer in Cluster (Connected)
```

## Configuración de un volumen GlusterFS

En este taller vamos a crear un volumen distribuido. En los tres nodos ejecutamos:

```
mkdir -p /data/brick1/gv0
```

Y en cualquier nodo creamos el volumen:

```
gluster volume create gv0 replica 3 nodo1:/data/brick1/gv0 nodo2:/data/brick1/gv0 nodo3:/data/brick1/gv0
```

Creamos un volumen gluster llamado `gv0`, con la opción `replica` hacemos que el volumen sea distribuido, es decir, cada servidor albergará una copia de los datos.Especificamos qué nodos usar, y qué bricks en esos nodos.

Podemos obtener información del volumen creado:

```
root@nodo1:~# gluster volume info
 
Volume Name: gv0
Type: Replicate
Volume ID: 34825f48-fe2b-4d0a-8c58-ba69cccb292e
Status: Created
Snapshot Count: 0
Number of Bricks: 1 x 3 = 3
Transport-type: tcp
Bricks:
Brick1: nodo1:/data/brick1/gv0
Brick2: nodo2:/data/brick1/gv0
Brick3: nodo3:/data/brick1/gv0
Options Reconfigured:
cluster.granular-entry-heal: on
storage.fips-mode-rchecksum: on
transport.address-family: inet
nfs.disable: on
performance.client-io-threads: off
```

Y por últimos iniciamos el volumen:

```
gluster volume start gv0
```

## Probando el volumen

Podemos montar el volumen en los propios nodos del cluster para ver cómo funciona. Por ejemplo podemos montar el volumen en el `nodo1`:

```
root@nodo1:~# mount -t glusterfs nodo1:/gv0 /mnt
root@nodo1:~# cd /mnt/
root@nodo1:/mnt# for i in `seq -w 1 100`; do cp -rp /var/log/messages /mnt/copy-test-$i; done
```

Y ahora en cualquiera de los nodos podemos comprobar cómo se han creado los 100 ficheros:

```
root@nodo3:/data/brick1/gv0# ls
```

O podríamos montar el volumen en otro nodo y ver los 100 ficheros creados:

```
root@nodo2:~# mount -t glusterfs nodo2:/gv0 /mnt
root@nodo2:~# cd /mnt/
root@nodo2:/mnt# ls
```

**Nota: Para que los ficheros se distribuyan hay que crearlo en un volumen montado, no se pueden crear directamente en el directorio del volumen `/data/brick/vg0`.**

## Probando el volumen desde un cliente externo

Para montar el volumen en un cliente externo, este debe conocer los nombres de los nodos del cluster, por lo tanto en su fichero `/etc/hosts` indicamos:

```
10.1.1.101 nodo1
10.1.1.102 nodo2
10.1.1.103 nodo3
```

A continuación instalamos el cliente de glusterfs:

```
apt install glusterfs-client
```

Y ya podemos montar el volumen (podemos indicar cualquiera de los 3 nodos):

```
root@cliente:~# mount -t glusterfs nodo1:/gv0 /mnt
root@cliente:~# cd /mnt/
root@cliente:/mnt# ls
```

¿Qué pasa si un nodo del cluster se apaga? Desde el cliente se sigue accediendo a la información del cluster. Podemos apagar un nodo y volvemos a comprobar que tenemos acceso a los ficheros.

## Conclusiones

Esto ha sido sólo una introducción. Para seguir avanzando habría que estudiar cómo se crear y se gestionan volúmenes de otros tipos, estudiar las [operaciones que se pueden realizar a los volúmenes](https://docs.gluster.org/en/latest/Administrator-Guide/Managing-Volumes/),...

Como inspiración para realizar este taller me he basado en la [Documentación oficial](https://docs.gluster.org) y en el artículo: [GlusterFS: crea tu almacenamiento distribuido](https://www.proxadmin.es/blog/glusterfs-almacenamiento-distribuido/).


