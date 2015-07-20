Документация к клиентской части TANGO
=====================================
.. Надо дописать некоторое введение. А именно, ответить на вопросы: 1. для чего нужна эта часть проекта; 2. за что она отвечает и 3. Количество и перечень файлов, в которых написан исходный код. Какие питоновские пакеты необходимо устанивить для работы с программой. Если возможно, то версию танго или версию pytango прописать.

В данной части представлено описание всех команд и атрибутов TANGO-устройства ``tomograph``. Посредством данного API можно управлять томографом по сети. Знакомство с модулем лучше начинать с :ref:`примера работы <tango-client-example>`.

Для понимания происходящего необходимо прочитать `пример использования PyTango <http://www.esrf.eu/computing/cs/tango/tango_doc/kernel_doc/pytango/latest/quicktour.html#client>`_ из официальной документации, а также может понадобиться `PyTango Client API <http://www.esrf.eu/computing/cs/tango/tango_doc/kernel_doc/pytango/latest/client_api/index.html>`_.

Установка
~~~~~~~~~

Linux
-----

Первый способ (сборка из исходников)

.. code-block:: bash

    pip install pytango --egg

Второй способ (быстрее, но более ранняя версия PyTango)

.. code-block:: bash

    sudo apt-get install python-pytango


Windows
-------

Установите следующие пакеты (или аналогичные им):

* `Python 2.7 <https://www.python.org/ftp/python/2.7.9/python-2.7.9.msi>`_
* `Numpy 1.9.2 <http://sourceforge.net/projects/numpy/files/NumPy/1.9.2/numpy-1.9.2-win32-superpack-python2.7.exe/download>`_
* `PyTango 8.1.6 <https://pypi.python.org/packages/2.7/P/PyTango/PyTango-8.1.6.Win32-py2.7.exe>`_


Подключение к томографу

.. code-block:: python

    tomograph = PyTango.DeviceProxy('188.166.73.250:10000/tomo/tomograph/1')

или прописать в ``~/.bashrc`` (Linux)

.. code-block:: bash
    
    export TANGO_HOST=188.166.73.250

и подключаться

.. code-block:: python

    tomograph = PyTango.DeviceProxy('tomo/tomograph/1')


Общие команды
~~~~~~~~~~~~~

.. function:: DevicesInfo()

   TODO

.. function:: SelfTest()

   На данный момент делает ping всех девайсов.


Двигатель
~~~~~~~~~

Атрибуты
--------

.. attribute:: angle_position

   Угод поворота двигателя в градусах.

   :type: float

.. attribute:: horizontal_position

   Положение двигателя по горизонтали.

   :type: int

.. attribute:: vertical_position

   Положение двигателя по вертикали.

   :type: int


Команды
-------

.. function:: MoveAway()

   Убирает объект из поля зрения детектора

.. function:: MoveBack()

   Возвращает объект в поле зрения детектора

.. function:: ResetAnglePosition()

   Делает текущий угол поворота новым нулем.

.. function:: MotorStatus()

   :rtype: str
   :returns: Возвращает JSON-строку следующего формата 

     .. code-block:: javascript

      {
        "state": текущее состояние двигателя: OFF, ON, MOVING (без префикса PyTango)  
        "angle position": угол поврота
        "horizontal position": позиция по горизонтали
        "vertical position": позиция по вертикали
      }


Источник рентгеновского излучения
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Атрибуты
--------

.. attribute:: xraysource_voltage

   Напряжение в кВ с точностью до десятых. 2,0 кВ <= voltage <= 60,0 кВ

   :type: float


.. attribute:: xraysource_current

   Ток в мА с точностью до десятых. 2,0 мА <= current <= 80,0 мА

   :type: float


Команды
-------

.. function:: PowerOn()

   Переводит источник рентгеновского излучения в состояние ON

.. function:: PowerOff()

   Переводит источник рентгеновского излучения в состояние OFF

.. function:: XRaySourceStatus()

   :rtype: str
   :returns: Возвращает JSON-строку следующего формата 

     .. code-block:: javascript

      {
        "model": Isovolt 3003
        "state": текущее состояние источника: OFF, ON, STANDBY, FAULT (без префикса PyTango)  
        "voltage": текущее значение напряжения
        "current": текущее значение тока
      }


Заслонка
~~~~~~~~

Команды
-------

.. function:: OpenShutter(time)

   Открывает заслонку на заданное время. Если time == 0, то открывает до вызова :func:`CloseShutter`

   :param time: Время в секундах, через которое нужно закрыть заслонку, или 0, если закрывать не нужно 
   :type time: float

.. function:: CloseShutter(time)

   Закрывает заслонку на заданное время. Если time == 0, то закрывает до вызова :func:`OpenShutter`

   :param time: Время в секундах, через которое нужно открыть заслонку, или 0, если открывать не нужно 
   :type time: float

Точность, с которой можно задавать time неизвестна. Однако, как говорит `StackOverflow <http://stackoverflow.com/questions/1133857/how-accurate-is-pythons-time-sleep>`_, можно рассчитывать на 50 мс.

.. function:: ShutterStatus()

   :rtype: str
   :returns: Возвращает JSON-строку следующего формата 

     .. code-block:: javascript

      {
        "state": текущее состояние двигателя: OPEN, CLOSE (без префикса PyTango)
      }


Детектор
~~~~~~~~

Команды
-------


.. attribute:: image

   После вызова :any:`GetFrame()` здесь лежит полученное изображение.

   .. warning::

      Этот атрибут у детектора, а не у томографа! Пока что.

   :type: PyTango.EncodedAttribute


.. function:: GetFrame(exposure)

   Получает изображение с детектора. Возвращает метаданные изображения. Само изображение лежит в атрибуте :any:`image`.

   :param exposure: Время экспозиции в 0,1 миллисекунд. 1 <= exposure (0,1 ms) <= 160000, т. е. от 0,1 миллисекунды до 16 секунд.
   :type exposure: int
   :rtype: str
   :returns: Возвращает JSON-строку следующего формата

     .. code-block:: javascript

      {
        "image_data": 
              {
                "exposure": время экспозиции
                "datetime": дата и время получения изображения в формате dd.mm.yyyy hh:mm:ss
                "detector": 
                      {
                        "model": модель детектора
                      }
              }
        "object": 
              {
                "present": True, если объект присутствует, и False иначе
                "angle position": угол поворота объекта
                "horizontal position": положение объекта по горизонтали
                "vertical position": положение объекта по вертикали
              }
        "shutter":
              {
                "open": True, если заслонка открыта, и False иначе
              }

        "X-ray source": 
              {
                "voltage": напряжение
                "current": ток
              }
      }

.. function:: DetectorStatus()

   :rtype: str
   :returns: Возвращает JSON-строку следующего формата 

     .. code-block:: javascript

      {
        "model": Ximea xiRAY
        "state": текущее состояние детектора: OFF, ON, RUNNING (без префикса PyTango)
        "exposure": текущее значение времени экспозиции
      } 


Состояния
---------

PyTango.DevState.OPEN

PyTango.DevState.CLOSE

PyTango.DevState.ON

PyTango.DevState.OFF

и т. д.

.. _tango-client-example:

Пример работы
~~~~~~~~~~~~~

.. code-block:: python
    :linenos:

    import PyTango
    PyTango.Release.version # '8.1.6'

    tomograph = PyTango.DeviceProxy('188.166.73.250:10000/tomo/tomograph/1')
    detector = PyTango.DeviceProxy('188.166.73.250:10000/tomo/detector/1')

    tomograph.xraysource_voltage = 40
    tomograph.xraysource_current = 20
    tomograph.PowerOn()
    tomograph.OpenShutter()
    tomograph.GetFrame(1000)
    # detector.image
    tomograph.CloseShutter()
    tomograph.PowerOff()


    ## Также можно подключиться к отдельным девайсам:
     
    shutter = PyTango.DeviceProxy('188.166.73.250:10000/tomo/shutter/1')
    shutter.Close()
    shutter.State() # PyTango._PyTango.DevState.CLOSE
    shutter.Status() # 'The device is in CLOSE state.'
    shutter.Open()
    shutter.State() # PyTango._PyTango.DevState.OPEN
     
    source = PyTango.DeviceProxy('188.166.73.250:10000/tomo/source/1')
    source.On()
    source.State() # PyTango._PyTango.DevState.ON
    source.Off()
    source.voltage = 10
    source.voltage # 10.0
     
    motor = PyTango.DeviceProxy('188.166.73.250:10000/tomo/motor/1')
    motor.angle_position = 1
    motor.angle_position # 1.0
    motor.ResetAnglePosition()
    motor.angle_position # 0.0

