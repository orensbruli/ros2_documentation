.. redirect-from::

  Contributing/Build-Farms

.. _BuildFarms:

===============
ROS Build Farms
===============

.. contents:: Tabla de contenidos
   :depth: 1
   :local:

Las granjas de compilación de ROS son una infraestructura importante para apoyar el ecosistema de ROS, proporcionada y mantenida por `Open Robotics`_.
Proporcionan la construcción de paquetes fuente y binarios, integración continua, pruebas y análisis para paquetes de ROS 1 y ROS 2.
Hay dos instancias alojadas para paquetes de código abierto:

#. https://build.ros.org/ para paquetes de ROS 1
#. https://build.ros2.org/ para paquetes de ROS 2

Si va a utilizar cualquiera de las infraestructuras proporcionadas, considere registrarse en el
`foro de discusión de la granja de compilación <http://discourse.ros.org/c/buildfarm>`__ para recibir notificaciones,
por ejemplo, sobre cualquier cambio próximo.

Trabajos y despliegue
-------------------

Las granjas de compilación de ROS realizan varios trabajos diferentes.
Para cada tipo de trabajo, encontrará una descripción detallada de lo que hacen y cómo funcionan:

* `trabajos de liberación`_ generan paquetes binarios, por ejemplo, paquetes debian
* `trabajos de desarrollo`_ construyen y prueban paquetes de ROS dentro de un solo repositorio sobre una base de encuesta
* `trabajos de solicitudes de extracción`_ construyen y prueban paquetes de ROS dentro de un solo repositorio desencadenado por webhooks
* `trabajos de integración continua`_ construyen y prueban paquetes de ROS en repositorios con la opción de usar artefactos
  de otros trabajos de integración continua para acelerar la construcción
* `trabajos de documentación`_ generan la documentación de la API de los paquetes y extraen información de los manifiestos
* `trabajos varios`_ realizan tareas de mantenimiento y generan datos informativos para visualizar el
  estado de la granja de compilación y sus artefactos generados

Creación y despliegue
.......................

Los trabajos anteriores se crean y se despliegan cuando se crean paquetes, es decir, liberados para ROS
1 o ROS 2.
Una vez que la floración es exitosa y un paquete se incorpora en una de las ROS
distribuciones (a través de una solicitud de extracción a rosdistro_), se generarán los trabajos correspondientes.
Los nombres de los trabajos codifican su tipo y propósito: [1]_

* trabajos de liberación:

   * ``{distro}src_{platf}__{package}__{platform}__source`` construir paquetes fuente de las liberaciones
   * ``{distro}bin_{platf}__{package}__{platform}__binary`` construir paquetes binarios de las liberaciones

   Por ejemplo, el trabajo de empaquetado binario de rclcpp en ROS 2 Humble (ejecutándose en Ubuntu Jammy amd64) se llama ``Hbin_uJ64__rclcpp__ubuntu_focal_amd64__binary``.

* trabajos de desarrollo:

   * ``{distro}dev__{package}__{platform}`` realiza una compilación CI para la rama de liberación.

* trabajos de solicitud de extracción

   * ``{distro}pr__{package}__{platform}`` realiza una compilación CI para una solicitud de extracción.

   Por ejemplo, el trabajo de PR para rclcpp en ROS 2 Humble (ejecutándose en Ubuntu Jammy amd64) se llama ``Hpr__rclcpp__ubuntu_jammy_amd64``.

Ejecución
.........

La ejecución de los trabajos depende del tipo de trabajo:

* Los trabajos de `desarrollo`_ se activarán cada vez que se realice un compromiso en la rama respectiva, en función de una frecuencia configurada.
* Los trabajos de `solicitud de extracción`_ se activarán mediante webhooks de la solicitud de extracción respectiva del repositorio aguas arriba [2]_
* Los trabajos de `lanzamiento`_ se activarán una vez cada vez que se publique una nueva versión del paquete, es decir, se aceptó una nueva solicitud de extracción de rosdistro_ para este paquete. Los trabajos de origen se activan mediante un cambio de versión en el archivo de distribución rosdistro, los trabajos binarios se activan mediante su homólogo de origen.


Preguntas frecuentes (FAQ) y solución de problemas
---------------------------------------------------

#. **Recibo correos electrónicos de Jenkins de trabajos fallidos en la granja de compilación. ¿Qué hago?**

   Vaya al trabajo que provocó el problema. Encontrará el enlace en la parte superior del correo electrónico de Jenkins.
   Una vez que haya seguido el enlace al trabajo de compilación, haga clic en *Salida de la consola* a la izquierda, y luego haga clic
   *Registro completo*. Esto le dará la salida de consola completa de la compilación fallida. Trate de encontrar el
   el error más importante, ya que suele ser el más importante y otros errores pueden ser secuelas.

   La parte inferior del correo electrónico puede leer ``'apt-src build [...]' failed. Esto suele ser debido a
   un error al construir el paquete.`` Esto suele indicar la falta de dependencias, consulte 2.

#. **Parece que me falta una dependencia, ¿cómo descubro cuál es?**

   Básicamente, tiene dos opciones, a. es más fácil pero puede requerir varias iteraciones, b. es más
   elaborado y le brinda una visión completa y también depuración local.

   a) Inspeccione el trabajo de lanzamiento que provocó el problema (consulte 1.) y localice la dependencia de cmake
      problema de dependencia. Para hacerlo, navegue hasta la sección cmake, por ejemplo, navegue hasta la sección *construir binarydeb*
      sección a través del menú de la izquierda en caso de un trabajo de compilación ubuntu / debian. El *Error de CMake*
      típicamente indicará una dependencia requerida por la configuración de CMake pero que falta en el
      `manifiesto del paquete`_. Una vez que haya solucionado la dependencia en el manifiesto, realice una nueva publicación
      de su paquete y espere comentarios de las granjas o...
   b) Para obtener una visión completa y una depuración local más rápida, se pueden `ejecutar los trabajos de lanzamiento localmente`_.
      Esto permite iterar localmente el manifiesto hasta que se arreglen todas las dependencias.

#. **¿Por qué los trabajos de lanzamiento fallan cuando los trabajos de desarrollo / mis acciones de github / mis compilaciones locales tienen éxito?**

   Hay varias posibles razones para esto.
   En primer lugar, los trabajos de lanzamiento se construyen con una instalación ROS mínima para comprobar si todas las dependencias están declaradas correctamente en el `manifiesto del paquete`_. Los trabajos de desarrollo / acciones de github / compilaciones locales se pueden realizar en un entorno en el que las dependencias ya están instaladas, por lo tanto, no se detectan problemas de dependencias. En segundo lugar, pueden construir diferentes versiones del código fuente. Mientras que los trabajos de desarrollo / acciones de github / compilaciones locales suelen construir la última versión del repositorio *upstream* [2]_, `los trabajos de lanzamiento`_ construyen el código fuente de la última versión de lanzamiento, es decir, el código fuente en las respectivas ramas *upstream* del repositorio de *lanzamiento* [3]_.


Lectura adicional
-----------------

Los siguientes enlaces proporcionan más detalles e información sobre las granjas de compilación:

* https://github.com/ros-infrastructure/ros_buildfarm/blob/master/doc/index.rst - Documentación general de la infraestructura de la granja de compilación y los trabajos de compilación generados.
* http://wiki.ros.org/regression_tests#Setting_up_Your_Computer_for_Prerelease
* http://wiki.ros.org/buildfarm - Entrada del wiki de ROS para la granja de compilación ROS 1 (parcialmente *desactualizado*).
* https://github.com/ros-infrastructure/cookbook-ros-buildfarm - Instala y configura máquinas de granja de compilación de ROS.


.. [1] ``{distro}`` es la primera letra de la distribución de ROS, ``{platform}`` (``{platf}``) nombra la plataforma para la que se construye el paquete (y su código corto), y ``{package}`` es el nombre del paquete ROS que se está construyendo.
.. [2] El repositorio *upstream* es el repositorio que contiene el código fuente original del paquete ROS 1 / ROS 2 correspondiente.
.. [3] El repositorio de *lanzamiento* es el repositorio que utiliza la infraestructura de ROS 2 para lanzar paquetes, consulte https://github.com/ros2-gbp/.

.. _`los trabajos de lanzamiento`:
   https://github.com/ros-infrastructure/ros_buildfarm/blob/master/doc/jobs/release_jobs.rst
.. _`los trabajos de desarrollo`:
   https://github.com/ros-infrastructure/ros_buildfarm/blob/master/doc/jobs/devel_jobs.rst
.. _`trabajos de solicitud de extracción`:
   https://github.com/ros-infrastructure/ros_buildfarm/blob/master/doc/jobs/devel_jobs.rst
.. _`trabajos de CI`:
   https://github.com/ros-infrastructure/ros_buildfarm/blob/master/doc/jobs/ci_jobs.rst
.. _`trabajos de documentación`:
   https://github.com/ros-infrastructure/ros_buildfarm/blob/master/doc/jobs/doc_jobs.rst
.. _`miscellaneous jobs`:
   https://github.com/ros-infrastructure/ros_buildfarm/blob/master/doc/jobs/miscellaneous_jobs.rst
.. _bloomed:
   http://wiki.ros.org/bloom
.. _rosdistro:
   https://github.com/ros/rosdistro
.. _`run the release jobs locally`:
   https://github.com/ros-infrastructure/ros_buildfarm/blob/master/doc/jobs/release_jobs.rst#run-the-release-job-locally
.. _`Open Robotics`:
   https://www.openrobotics.org/
.. _`job descriptions above`:
   #jobs-and-deployment
.. _`package manifest`:
   http://wiki.ros.org/Manifest
