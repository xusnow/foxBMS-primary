Isoguard
========

.. include:: ../../../macros.rst

.. highlight:: C

The |isoguard module| measures the insulation resistance between the insulated 
and active HV-conductors of the battery and the reference earth (chassis ground/ Kl.31).

.. ignore .. sectnum::

.. contents:: Table Of Contents



Module Files
~~~~~~~~~~~~

Driver:
 - src\\engine\\isoguard\\isoguard.h
 - src\\engine\\isoguard\\isoguard.c
 - src\\engine\\isoguard\\ir155.h
 - src\\engine\\isoguard\\ir155.c
 
Driver Configuration:
 - src\\engine\\config\\isoguard_cfg.h
 - src\\engine\\config\\isoguard_cfg.c



Dependencies
~~~~~~~~~~~~
CPU
 - src\\module\\cpu\\cpu.h

Database
 - src\\engine\\database\\database.h
 - src\\engine\\config\\database_cfg.h

Diag
 - src\\engine\\diag\\diag.h
 - src\\engine\\config\\diag_cfg.h

IO
 - src\\module\\io\\io.h
 - src\module\\config\\io_cfg.h

RTC
 - src\\module\\rtc\\rtc.h
 - src\\module\\config\\rtc_cfg.h
 
TIMER
 - src\\module\\timer\\timer.h
 - src\\module\\config\\timer_cfg.h

Detailed description of Isoguard-Module
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

File Structure and Interfaces
-----------------------------

The |isoguard module| is separated into two different layers, a bottom and a top layer. The top layer 
consists of the ``isoguard.c`` and its associated ``isoguard.h`` file. It provides an easy interface 
to initialize the |isoguard module| and to access the measurement data. The insulation threshold to differentiate 
between a good and bad insulation can be set in the ``isoguard_cfg.h`` file. The bottom layer contains 
the ``ir155.c`` and ``ir155.h`` files. They are both dedicated to the Bender ISOMETER IR155-3204 
hardware and handle the connection of the ISOMETER to the MCU, as well as the evaluation and interpretation of 
the measurement data.

The external interface to the |isoguard module| is simple and consists of three functions and is 
viewable in :numref:`fig. %s <isoguard_figure1>`:

.. _isoguard_figure1:
.. figure:: ISO_Block.png
   :height: 286 px
   :width: 1042 px
   :scale: 55 %

   External Isoguard Driver interface

``ISO_Init(void)``
   Initializes SW-|isoguard module| and Enables HW-Bender-module
``ISO_ReInit(void)``
   Resets SW-|isoguard module| and HW-Bender-module
``ISO_MeasureInsulation(void)`` 
   Measures and evaluates the insulation

Configuration
-------------
The configuration is done in the ``isoguard_cfg.h`` file and consists of two defines. 

.. code-block:: C

   #define ISO_CYCLE_TIME                    200

``ISO_CYCLE_TIME`` is the periodic calling time of the ``ISO_MeasureInsulation(void)`` function and must not be chosen lower than 150ms
due to the lowest possible frequency of the Bender ISOMETER of 10Hz. The default value is 200ms. 

.. code-block:: C

   #define ISO_RESISTANCE_THRESHOLD          400
    
``ISO_RESISTANCE_THRESHOLD`` specifies the resistance threshold to differentiate between good and bad insulation. This value has no impact if 
the threshold is set lower than the intern resistance threshold of the Bender ISOMETER. The default value is 400 kOhm.

 
Initialization
--------------
The initialization is done via the interface function ``ISO_Init(void)``, which is forwarded to the Bender ISOMETER specific initialization functions,
as shown in :numref:`fig. %s <isoguard_figure2>`.

.. _isoguard_figure2:
.. figure:: ISO_Init.png
   :height: 500 px
   :width: 1293 px
   :scale: 55 %

   Initialization of |isoguard module|

The most important steps in the initialization process are:

 - Enabling of PWM Input measurement
 - Reading of BKP_SRAM variable for previous Ground Error
 - Set start-up time before measurement results are trustworthy (dependent of previous state)
 - Enabling of Hardware Bender Module

The function ``ISO_ReInit(void)`` resets the whole |isoguard module| and the initialization process is done again, including the waiting time until the
measurement values are declared as trustworthy.


Usage
~~~~~
After initializing the |isoguard module|, the ``ISO_MeasureInsulation(void)`` function needs to be called periodically according to the set cycle time. 
The Bender ISOMETER measurement values are evaluated and then written into the database where further modules can operate on this data. Following 
measurement values are saved:

.. code-block:: C

   typedef struct {
       uint8_t valid;         // 0 -> valid, 1 -> resistance unreliable
       uint8_t state;         // 0 -> resistance/measurement OK , 1 -> resistance too low or error
       uint8_t resistance;    // in intervals (max 3bit)
       uint32_t timestamp;
       uint32_t previous_timestamp;
   }DATA_BLOCK_ISOMETER_s;

The measured insulation is split into intervals according to the array ``uint16 const static ir155_ResistanceInterval[7]``. For more detailed 
information see source code.


References
~~~~~~~~~~

.. [1] Datasheet ISOMETER IR155-3204 [PDF] https://www.bender-de.com/fileadmin/products/doc/IR155-42xx-V004_DB_en.pdf
