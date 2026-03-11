.. container:: titlepage

   .. image:: imagenes/output-onlinepngtools.png
      :alt: Logo CUNQA
      :width: 95.0 %
      :align: center

   .. rst-class:: big-title

   Manual de Uso de CUNQA 2.0

   .. container:: logo-container

      .. image:: imagenes/logotipo_CESGA_original.jpg
         :alt: Logo CESGA

      .. image:: imagenes/COMPUTAEX_nuevo.png
         :alt: Logo COMPUTAEX

Introducción
============

CUNQA es un emulador de Computación Cuántica Distribuida (DQC) diseñado
nativamente para entornos HPC. Permite a los investigadores probar
algoritmos distribuidos (usando vQPUs o Virtual QPUs) antes de que el
hardware real esté disponible, integrándose transparentemente con el
gestor de colas SLURM.

.. note::

   Nota sobre la Documentación y Código Fuente Este documento constituye
   un **manual básico de inicio rápido** adaptado específicamente para
   el entorno del clúster **Lusitania**.

   La versión desplegada en este clúster cuenta con parches de
   compatibilidad específicos y se encuentra alojada en un *fork* del
   repositorio original, disponible en:

   .. container:: center

      https://github.com/Cerrudoxx/cunqa/tree/cunqa-ex-module-2

   Para consultar la documentación completa, detallada y oficial de
   CUNQA desarrollada por el CESGA, visite el siguiente enlace:

   .. container:: center

      https://cesga-quantum-spain.github.io/cunqa/index.html

Carga del Módulo en Lusitania
=============================

Para utilizar la librería CUNQA en Lusitania, es necesario cargar el
entorno y sus dependencias de compilación.

Carga Estándar (Sesión Limpia)
------------------------------

Utilice el siguiente comando para cargar los módulos:

.. code-block:: bash

   module load python/cunqa libraries/libffi-devel-8.1

.. warning::

   Aviso Importante: Parche Temporal Para que Python detecte
   correctamente el framework CUNQA y sus dependencias internas, es
   necesario configurar la variable de entorno ``PYTHONPATH``.
   Actualmente estamos trabajando para que esta ruta se integre de
   manera automática al cargar el módulo; mientras tanto, es necesario
   aplicar este comando como parche temporal:

Resolución de Conflictos (Module Swap)
--------------------------------------

CUNQA requiere una versión específica de Python. Si usted ya tiene
cargado otro módulo (ej. ``python/3.10``), Lmod mostrará un error de
conflicto. Utilice ``swap`` para intercambiarlos:

.. code-block:: bash

   # Si tiene conflictos con otra versión de Python module swap
   python/python-version-actual python/cunqa

Gestión de Recursos y Ciclo de Vida
===================================

El flujo de trabajo en CUNQA consta de tres pasos: Iniciar Servicio
(``qraise``), Ejecutar Experimentos (Python) y Detener Servicio
(``qdrop``).

Paso 1: Iniciar el Servicio (qraise)
------------------------------------

El comando ``qraise`` solicita recursos a SLURM y levanta los daemons de
simulación. En este ejemplo se solicitan 2 vQPUs durante 30 min:

.. code-block:: bash

   qraise -n 2 -t 00:30:00 --co-located

Tabla Completa de Parámetros
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Opciones disponibles en ``qraise`` para el despliegue de vQPUs:

.. list-table:: 
   :widths: 15 30 55
   :header-rows: 1

   * - Flag
     - Opción Larga
     - Descripción
   * - ``-n``
     - ``--num_qpus <int>``
     - (Obligatorio) Número de vQPUs a levantar (Defecto: 0).
   * - ``-t``
     - ``--time <str>``
     - (Obligatorio) Tiempo límite de la reserva (HH:MM:SS).
   * - 
     - ``--family_name <str>``
     - Nombre para identificar el grupo de vQPUs (Defecto: "default").
   * - 
     - ``--co-located``
     - **Esencial en Lusitania.** Permite acceder a las vQPUs desde cualquier nodo (ej. el nodo de login).
   * - ``-c``
     - ``--cores <int>``
     - Número de cores de CPU asignados a cada vQPU (Defecto: 2).
   * - ``-p``
     - ``--partition <str>``
     - Partición de SLURM solicitada para el despliegue.
   * - 
     - ``--mem-per-qpu <int>``
     - Cantidad de memoria RAM (en GB) asignada a cada vQPU.
   * - ``-N``
     - ``--n_nodes <int>``
     - Número total de nodos de cómputo a utilizar (Defecto: 1).
   * - 
     - ``--node_list <str>``
     - Lista explícita de nodos destino (ej. ``s01r2b12,s01r2b13``).
   * - 
     - ``--qpus_per_node <int>``
     - Número de vQPUs a desplegar dentro de cada nodo físico.
   * - 
     - ``--gpu``
     - Habilita la simulación cuántica acelerada por GPU.
   * - 
     - ``--classical_comm``
     - Habilita las comunicaciones clásicas entre vQPUs.
   * - 
     - ``--quantum_comm``
     - Habilita las comunicaciones cuánticas entre vQPUs.
   * - ``-sim``
     - ``--simulator <str>``
     - Motor responsable de ejecutar la simulación (Defecto: "Aer").
   * - ``-b``
     - ``--backend <str>``
     - Ruta al archivo de configuración del backend.
   * - ``--noise-prop``
     - ``--noise-properties <str>``
     - Ruta al JSON de propiedades de ruido (Solo soportado en Aer).
   * - ``-fq``
     - ``--fakeqmio <str>``
     - Simula el chip QMIO cargando un set de calibraciones (*Nota: Soportado oficialmente en CESGA*).
   * - 
     - ``--no-thermal-relaxation``
     - Desactiva la relajación térmica en backends ruidosos (Defecto: false).
   * - 
     - ``--no-readout-error``
     - Desactiva el error de lectura en backends ruidosos (Defecto: false).
   * - 
     - ``--no-gate-error``
     - Desactiva el error de puertas en backends ruidosos (Defecto: false).
   * - 
     - ``--qmio``
     - Despliega infraestructura híbrida conectando con el ordenador cuántico QMIO real (*Exclusivo de CESGA*).

Selección de Partición (``sinfo``)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

En el clúster Lusitania, los nodos de cómputo están organizados en
diferentes colas o particiones. Puede especificar en qué partición desea
desplegar sus vQPUs utilizando el flag ``-p`` o ``–partition`` al
ejecutar el comando ``qraise``.

Para consultar las particiones disponibles, los recursos libres y el
estado general del clúster, ejecute el comando:

.. code-block:: bash

   sinfo

**Ejemplo de salida esperada:**

.. code-block:: bash

   PARTITION   AVAIL  TIMELIMIT  NODES  STATE NODELIST
   qcomputaex     up   infinite      1  alloc s02r2b05
   lusi2          up   infinite      4 drain* s02r2b[51-54]
   lusi2          up   infinite    108   idle s02r1b[43-83],s02r2b[11-50,56-57,60-84]
   atoms          up   infinite     14   idle s02r1b[01-12,15-16]
   lusitania* up   infinite     14   idle s01r3b[03-16]
   uex10          up   infinite      9  alloc s01r2b[02-10]
   fatnode        up   infinite      1    mix fatcn

**Interpretación de las columnas principales:**

-  **PARTITION:** Nombre de la cola de ejecución. El asterisco (``*``)
   indica cuál es la partición por defecto (en este caso,
   ``lusitania``). Si no especifica el flag ``-p``, su trabajo se
   enviará automáticamente a esta cola.

-  **AVAIL:** Estado operativo de la partición (``up`` significa que
   está activa y admitiendo trabajos).

-  **NODES:** Cantidad de nodos físicos que se encuentran en el estado
   indicado.

-  **STATE:** Estado actual de los nodos. Los valores más comunes son:

   -  ``idle``: Libres y listos para ejecutar simulaciones.

   -  ``alloc``: Totalmente ocupados por otros trabajos.

   -  ``mix``: Parcialmente ocupados (aún tienen algunos cores o memoria
      libre).

   -  ``drain``: Fuera de servicio o reservados para mantenimiento.

Monitorización de vQPUs (``squeue`` y ``qinfo``)
------------------------------------------------

Una vez solicitado el despliegue mediante el comando ``qraise``, el
proceso de monitorización para confirmar que todo está listo consta de
dos pasos fundamentales: verificar la asignación en el gestor de colas y
consultar el estado interno de CUNQA.

Paso 1: Verificación de recursos en SLURM (``squeue``)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Antes de ejecutar su algoritmo en Python, es estrictamente necesario
confirmar que el clúster le ha asignado los recursos y el trabajo ha
comenzado. Para ello, compruebe la cola de trabajos:

.. code-block:: bash

   squeue

**Salida esperada:**

.. code-block:: bash

   JOBID   PARTITION  NAME    USER      ST  TIME  NODES NODELIST(REASON)
   718298  lusitania  qraise  usuario   R   0:05  1     s01r3b18

Debe prestar especial atención a dos columnas:

-  **ST (Status):** Su trabajo debe marcar **R** (*Running*). Si marca
   **PD** (*Pending*), significa que está en la cola esperando a que
   haya nodos libres.

-  **NODELIST:** Esta columna revela el nombre del nodo físico exacto
   donde se han levantado sus vQPUs (en el ejemplo superior,
   ``s01r3b18``). Anote este valor para el siguiente paso.

Paso 2: Inspección interna de CUNQA (``qinfo``)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Una vez que el trabajo está en estado ``R``, puede emplear la
herramienta nativa ``qinfo`` para obtener información detallada sobre
las vQPUs que están corriendo bajo el capó (puertos abiertos, simulador
empleado, etc.).

**Opciones de consulta:**

-  ``qinfo <nodo>``: Muestra el estado de las vQPUs alojadas en el nodo
   físico especificado por parámetro.

-  ``qinfo -m`` o ``--mynode``: Muestra el estado de las vQPUs locales al
   nodo donde se ejecuta el comando (muy útil si se utiliza dentro de un
   script *sbatch* o tras acceder directamente al nodo de cómputo).

**Ejemplo de uso:** Utilizando el nombre del nodo que descubrimos en el
paso anterior (``s01r3b18``), ejecutamos desde el nodo de *login*:

.. code-block:: bash

   qinfo s01r3b18

Logs y Depuración
-----------------

Al solicitar un despliegue con ``qraise``, el gestor de colas SLURM
captura toda la salida estándar (mensajes de inicialización de CUNQA) y
los posibles errores del sistema, guardándolos automáticamente en un
archivo de texto. Este archivo se crea en el mismo directorio desde
donde ejecutó el comando.

El archivo generado sigue la nomenclatura ``qraise_<JOBID>``. Para
visualizar su contenido y comprobar que las vQPUs han arrancado
correctamente, utilice el comando ``cat`` (sustituyendo el número por su
identificador real):

.. code-block:: bash

   cat qraise_718298

**Salida esperada:**

.. code-block:: bash

   (03/11/26 10:00:00 AM) [Executor 718298] debug: Raising executor with Aer.
   (03/11/26 10:00:01 AM) [QPU 718298_0] debug: Raising QPU with quantum communications.
   (03/11/26 10:00:01 AM) [QPU 718298_0] debug: Server bound to tcp://10.2.1.38:38105
   (03/11/26 10:00:01 AM) [QPU 718298_1] debug: Server bound to tcp://10.2.1.38:42083

**¿Qué debemos buscar en este archivo?**

-  **Inicialización (``Raising...``):** Confirma el motor de simulación
   que se está utilizando (ej. ``Aer``) y si los protocolos de
   comunicación (cuántica o clásica) se han activado correctamente.

-  **Apertura de puertos (``Server bound to...``):** Es la señal
   definitiva de éxito. Indica la dirección IP interna y el puerto por
   donde la QPU virtual está escuchando instrucciones. **Una vez vea
   estas líneas, las vQPUs están listas para recibir comandos desde su
   script de Python.**

-  **Errores de SLURM:** Si el trabajo falla prematuramente (por
   ejemplo, por solicitar más cores de los permitidos o por falta de
   librerías dinámicas), los errores fatales de ``srun`` aparecerán en
   las primeras líneas de este log.

.. note::

   Truco de Experto: Monitorización en tiempo real Si el clúster está
   muy saturado o el trabajo tarda en arrancar, puede monitorizar la
   creación del log en tiempo real (como si fuera una pantalla de
   terminal en vivo) utilizando el comando ``tail``. Para salir de esta
   vista interactiva, simplemente pulse ``Ctrl + C``:

.. code-block:: bash

   tail -f qraise_718298

Liberación de Recursos (``qdrop``)
----------------------------------

El comando ``qdrop`` se utiliza para terminar (*drop*) las QPUs
virtuales que fueron desplegadas previamente con ``qraise``. Al ejecutar
este comando, se cancelan los trabajos asociados en SLURM y se liberan
los recursos computacionales del clúster.

Aunque el servicio se detiene automáticamente al alcanzar el tiempo
límite (``-t``) asignado por SLURM, es una **buena práctica en entornos
HPC** liberar los recursos manualmente en cuanto su algoritmo termine de
ejecutarse para que otros usuarios puedan utilizarlos.

**Sinopsis del comando:**

.. code-block:: bash

   qdrop [IDS...] [OPTIONS]

**Opciones de selección de objetivos:**

-  ``IDS...``: IDs de los trabajos de SLURM que se desean eliminar. Se
   pueden proporcionar múltiples IDs separados por espacios.

-  ``–fam``, ``--family_name <str>`` Elimina todas las vQPUs que
   pertenezcan a la familia especificada (útil si usó el flag ``-fam``
   al levantarlas).

-  ``--all``: Elimina todos los trabajos activos de ``qraise`` del
   usuario actual.

Ejemplos de Uso
~~~~~~~~~~~~~~~

**Opción A: Detener trabajos específicos (por ID)** Puede detener un
único trabajo o varios a la vez pasando sus identificadores separados
por un espacio:

.. code-block:: bash

   qdrop 718298 718299

**Salida esperada:**

.. code-block:: bash

   Removed job(s) with ID(s): 718298, 718299

**Opción B: Detener por familia** Si levantó un conjunto de vQPUs bajo
una misma etiqueta, puede borrarlas todas de golpe sin necesidad de
buscar sus IDs:

.. code-block:: bash

   qdrop --family_name grover_est

**Opción C: Limpieza total (``--all``)** El método más rápido y
recomendado si ya ha terminado su sesión de trabajo y quiere asegurarse
de no dejar ningún proceso "fantasma" consumiendo horas de cómputo en
Lusitania:

.. code-block:: bash

   qdrop --all

.. note::

   Nota sobre filtros combinados Si se proporcionan múltiples selectores
   a la vez (por ejemplo, especificando un ``ID`` y además una
   ``--family_name``), el omando ``qdrop`` intentará eliminar **todas**
   las vQPUs que coincidan con cualquiera de los criterios indicados.

Ejecución de Experimentos
=========================

CUNQA permite dos flujos de trabajo principales para desplegar las vQPUs
y ejecutar sus circuitos cuánticos: un enfoque mixto (desplegando desde
la terminal y ejecutando en Python) y un enfoque 100% programático desde
Python.

.. note::

   ¡Atención: Configuración Crítica en Lusitania! En el clúster
   Lusitania, es **obligatorio** que las vQPUs se levanten en modo
   *co-located*.

   Dado que SLURM asignará los recursos a nodos de cómputo distintos al
   nodo de *login* donde usted ejecuta el script, debe usar
   ``--co-located`` en la terminal y el parámetro ``co_located=True``
   dentro de la función ``get_QPUs()``. Si no lo hace, su script de
   Python no encontrará las máquinas virtuales.

Flujo 1: Despliegue Mixto (Bash + Python)
-----------------------------------------

Este es el método más recomendado, ya que separa la reserva de recursos
en SLURM de la lógica del algoritmo.

**1. Levantar las vQPUs desde la terminal:**

.. code-block:: bash

   qraise -n 4 -t 01:00:00 --co-located

**2. Ejecutar el código Python:**

.. code-block:: python
   :caption: demo_mixto.py

   from cunqa.qpu import get_QPUs, run
   from cunqa.qjob import gather
   from cunqa.circuit import CunqaCircuit

   # 1. Obtener las QPUs levantadas previamente por bash (Critico: co_located=True)
   qpus = get_QPUs(co_located=True)

   # 2. Definir un circuito de entrelazamiento (Bell State)
   qc = CunqaCircuit(num_qubits=2)
   qc.h(0)
   qc.cx(0,1)
   qc.measure_all()

   # 3. Enviar el mismo circuito a las 4 vQPUs a la vez
   qcs = [qc] * 4
   qjobs = run(qcs, qpus, shots=1000)

   # 4. Recopilar e imprimir los resultados
   results = gather(qjobs)
   for i, result in enumerate(results):
       print(f"vQPU {i} -> Counts: {result.counts}")

Flujo 2: Despliegue 100% en Python
----------------------------------

En este caso, la reserva de recursos en SLURM, la ejecución y la
liberación de los mismos se realiza de principio a fin dentro del mismo
script de Python.

.. code-block:: python
   :caption: demo_python.py

   from cunqa.qpu import qraise, get_QPUs, run, qdrop
   from cunqa.qjob import gather
   from cunqa.circuit import CunqaCircuit

   # 1. Desplegar QPUs directamente desde Python
   print("Solicitando recursos a SLURM...")
   family = qraise(4, "00:10:00", simulator="Aer", co_located=True)

   # 2. Conectar con las QPUs reservadas
   qpus = get_QPUs(co_located=True)

   # 3. Ejecutar circuito
   qc = CunqaCircuit(num_qubits=2)
   qc.h(0); qc.cx(0,1); qc.measure_all()
   qjobs = run([qc]*4, qpus, shots=1000)

   # 4. Mostrar resultados
   for result in gather(qjobs):
       print(f"Counts: {result.counts}")

   # 5. Liberar los recursos de SLURM
   print("Liberando recursos...")
   qdrop(family)

**Resumen del Ciclo de Vida Correcto:** Sea cual sea el flujo que elija,
es fundamental respetar el orden secuencial de operaciones para evitar
recursos huérfanos o errores de conexión en el clúster:

#. **Reserva:** Iniciar el entorno solicitando los nodos a SLURM
   (``qraise``).

#. **Verificación:** Si usa el flujo mixto, asegurarse mediante
   ``squeue`` de que el trabajo ha pasado a estado ``R`` (Running) y los
   puertos están abiertos.

#. **Ejecución:** Lanzar la lógica de su circuito (``run`` y
   ``gather``).

#. **Finalización:** Apagar las vQPUs y liberar los nodos físicos con
   ``qdrop``.

Capacidades Avanzadas
=====================

CUNQA permite emular arquitecturas de computación distribuida complejas.
Para ello, es necesario iniciar las vQPUs con los flags adecuados y
utilizar la API extendida de la clase ``CunqaCircuit``.

Comunicaciones Clásicas (CC)
----------------------------

Este modelo permite a los procesos intercambiar información clásica
durante la ejecución (de forma similar a como operan los programas MPI
en entornos HPC tradicionales). En el contexto cuántico, se utiliza para
enviar los resultados de medidas (bits clásicos) de una vQPU remota a
otra.

Esto es fundamental para algoritmos que requieren *feed-forward* (tomar
decisiones dinámicas sobre operaciones cuánticas locales basadas en los
resultados de una medida remota), como ocurre en la Teleportación
Cuántica o en los códigos de Corrección de Errores.


.. warning::
   **Requisito de Infraestructura**

   Para habilitar los canales de comunicación clásicos entre nodos (vía ZeroMQ), es **obligatorio**
   iniciar ``qraise`` con el flag ``--classical_comm`` en la terminal (o
   ``classical_comm=True`` si levanta las vQPUs desde Python). Además,
   recuerde mantener siempre activo el modo ``co_located``.

**Primitivas de la API de CUNQA 2.0:** Para poder condicionar
operaciones en base a bits remotos, debe asignar un ``id`` único a cada
circuito y seguir estos tres pasos usando los métodos de la clase
``CunqaCircuit``:

-  **Medir y Enviar (``send``):** Tras medir un qubit y guardar el valor
   en un bit clásico local, utilice ``send(clbits, recving_circuit)``
   para transmitir ese bit al circuito destino.

-  **Recibir (``recv``):** En el circuito destino, utilice
   ``recv(clbits, sending_circuit)`` para capturar la transmisión y
   almacenarla en el registro clásico local. *Nota: La ejecución de este
   circuito se bloqueará hasta que el bit sea recibido.*

-  **Control Condicional (``cif``):** Utilice el bloque de contexto
   ``with circuit.cif(clbits) as subcircuit:`` para aplicar operaciones
   cuánticas (ej. una puerta X) únicamente si el valor del bit clásico
   recibido es 1.

.. code-block:: python
   :caption: Ejemplo: Control Clásico Remoto entre dos vQPUs

   from cunqa.qpu import get_QPUs, qraise, qdrop, run
   from cunqa.circuit import CunqaCircuit
   from cunqa.qjob import gather

   # 1. Reservar 2 vQPUs con comunicaciones clasicas habilitadas
   print("Solicitando nodos a SLURM...")
   family = qraise(2, "00:10:00", classical_comm=True, co_located=True)
   qpus = get_QPUs(family=family, co_located=True)

   # 2. Definir los circuitos y sus IDs (vital para el enrutamiento de red)
   circuit_1 = CunqaCircuit(num_qubits=1, num_clbits=1, id="circuit_1")
   circuit_2 = CunqaCircuit(num_qubits=1, num_clbits=1, id="circuit_2")

   # --- LOGICA DEL EMISOR (circuit_1) ---
   circuit_1.h(0)
   circuit_1.measure(qubit=0, clbit=0)
   circuit_1.send(clbits=0, recving_circuit="circuit_2")

   # --- LOGICA DEL RECEPTOR (circuit_2) ---
   # Recibe el bit de circuit_1 y lo guarda en su clbit 0 local
   circuit_2.recv(clbits=0, sending_circuit="circuit_1")

   # Aplica una puerta X en su qubit 0 solo si el clbit 0 recibido vale 1
   with circuit_2.cif(clbits=0) as subcircuit:
       subcircuit.x(0)

   circuit_2.measure(qubit=0, clbit=0)

   # 3. Ejecucion distribuida
   # CUNQA mapea automaticamente circuit_1 a qpus y circuit_2 a qpus
   distributed_qjobs = run([circuit_1, circuit_2], qpus, shots=1000)
   results = gather(distributed_qjobs)

   for qpu, result in zip(qpus, results):
       print(f"Resultados de vQPU {qpu.id}: {result.counts}")

   # 4. Limpieza de recursos
   qdrop(family)

Comunicaciones Cuánticas (QC)
-----------------------------

Este paradigma es exclusivo de la computación cuántica distribuida y no
tiene contraparte clásica. Habilita la transferencia de información
cuántica (estados superpuestos o entrelazados) entre diferentes vQPUs
mediante el consumo de pares entrelazados (EPR pairs) compartidos
previamente.

.. warning::
   **Requisito de Infraestructura**
   
   Para utilizar estos protocolos, es **obligatorio** iniciar ``qraise`` con el flag ``--quantum_comm`` (o
   ``quantum_comm=True`` desde Python).

   *Nota:* Al habilitar las comunicaciones cuánticas, las directivas de
   comunicación clásica (CC) detalladas en la sección anterior quedan
   automáticamente permitidas en sus circuitos. Recuerde mantener
   siempre el modo ``co_located``.

Teledata (Teletransportación de Estado)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

El protocolo *Teledata* mueve el estado cuántico exacto de un qubit
desde una vQPU hacia otra. Debido al teorema de no clonación de la
mecánica cuántica, el estado en el qubit de origen se destruye y **se
resetea automáticamente al estado :math:`|0\rangle`** tras el envío.

En la API de CUNQA, esto se implementa mediante las primitivas ``qsend``
y ``qrecv``:

.. code-block:: python
   :caption: Ejemplo: Construcción remota de un estado entrelazado (Teledata)

   from cunqa.qpu import get_QPUs, qraise, qdrop, run
   from cunqa.circuit import CunqaCircuit
   from cunqa.qjob import gather

   # 1. Despliegue con comunicaciones cuanticas
   family = qraise(2, "00:10:00", quantum_comm=True, co_located=True)
   qpus = get_QPUs(family=family, co_located=True)

   # 2. Diseño de circuitos
   circuit_1 = CunqaCircuit(num_qubits=2, id="circuit_1")
   circuit_2 = CunqaCircuit(num_qubits=2, id="circuit_2")

   # Entrelazamos los qubits locales del circuito 1
   circuit_1.h(0)
   circuit_1.cx(0,1)

   # --- INICIO PROTOCOLO TELEDATA ---
   # Teleportamos el estado del qubit 1 (del circuit_1) al qubit 0 (del circuit_2)
   circuit_1.qsend(1, "circuit_2")
   circuit_2.qrecv(0, "circuit_1")
   # --- FIN PROTOCOLO TELEDATA ---

   # Continuamos operando en el circuito destino
   circuit_2.cx(0,1)

   circuit_1.measure_all()
   circuit_2.measure_all()

   # 3. Ejecucion y Limpieza (gather bloquea hasta terminar ambas simulaciones)
   qjobs = run([circuit_1, circuit_2], qpus, shots=1000)
   for qpu, result in zip(qpus, gather(qjobs)):
       print(f"Resultados de vQPU {qpu.id}: {result.counts}")

   qdrop(family)

Telegate (Puertas Lógicas Distribuidas)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

El protocolo *Telegate* permite ejecutar una puerta de dos qubits (como
una CNOT) donde el qubit de control reside físicamente en una vQPU y el
qubit objetivo (*target*) en otra.

A nivel de código, esto se logra "exponiendo" un qubit local para que un
circuito remoto pueda referenciarlo temporalmente:

.. code-block:: python
   :caption: Ejemplo: Puerta CNOT Distribuida (Telegate)

   circuit_A = CunqaCircuit(num_qubits=1, id="circuito_A")
   circuit_B = CunqaCircuit(num_qubits=2, id="circuito_B")

   circuit_A.h(0)

   # QPU A tiene el control (qubit 0) y lo expone a la QPU B
   with circuit_A.expose(qubit=0, target_circuit="circuito_B") as ctrl_ref:
       
       # Dentro de este bloque, 'ctrl_ref' es un enlace cuantico valido para B
       # Aplicamos una CNOT controlada por A y con target en el qubit 1 de B
       circuit_B.cx(control=ctrl_ref, target=1)

   circuit_A.measure_all()
   circuit_B.measure_all()

**Nota sobre el rendimiento (Execution):** Cuando se solicitan
comunicaciones cuánticas, las vQPUs desplegadas comparten internamente
un simulador conjunto por debajo (ZeroMQ/MPI). Por lo tanto, la llamada
a ``gather()`` es estrictamente bloqueante y esperará a que toda la
simulación global haya concluido para devolver los resultados y los
tiempos de ejecución (``result.time_taken``).