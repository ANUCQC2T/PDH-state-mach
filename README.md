PDH-state-mach
==============

State machine architecture used to control and phase-lock optical cavities and optical beam paths.

Brief description:
This digital control system was initially designed to control 10 phase locks:
4 of which were cavity-based, and 6 of which were interference-based. Further, there is a TTL
pulse generator that outputs 4 precisely timed TTL pulses, in sync with the phase locks. The
user interface gives access to these 10 locks and 4 TTL pulses on one screen complete with an
oscilloscope. The TTLs are used to switch various electronic equipment in the experiment.

Hardware and software structure:
The software package used was National Instruments LabVIEW 2010 (service pack add-on), operating
on Windows 7, 64-bit, on a desktop PC. 2 FPGA cards were used (NI PXI-7853R) installed
on a National Instruments 5-slot PXI chassis (NI PXI-1033). Custom-made breakout boxes for
BNC inputs and outputs were employed.

FPGA level VIs:
There are two FPGA level VIs. One controls 4 cavities, and is named ‘Cavities FPGA.vi’. The
other is named ‘Graph FPGA.vi’ and controls the phase locks necessary to create an entangled
graph state. These FPGA level VIs perform most of the logic that is required to control the
different phase locks, and a top level VI (detailed later) provides the user interface that allows
access to the various FPGA level controls in a user friendly way.

The FPGA level VI has 4 parallel while loops - Control loop, Oscilloscope loop, Pulse generator
loop, and a timing synchronisation loop. The most important of these is the Control Loop, shown
in Fig. 4.10. I will treat the other 3 loops in section 4.3.4. The functionality of the Control loop
should be apparent upon inspection. There are 4 subVIs arranged in a line near the top of the
loop. As is good LabVIEW coding practice, information here only ever flows left to right (or up
and down) and never right to left.

The Input subVI reads in data from the photodetectors on the lab table via an FPGA card. This
input (after being scaled) is fed to the Thresholds subVI which performs various tests in order to
tell if the cavity is on resonance or not - determined by being above or below a certain voltage
threshold. The State subVI will decide on the desired state of each cavity from threshold tests and
user inputs. The Output subVI then applies the necessary voltage to each cavity’s PZT mirror,
via the FPGA outputs and external amplifiers. In between the State subVI and Output subVI we
see that there are 4 case structures, 1 for each cavity.

Each case structure determines the state of each of the cavities, as dictated by the State VI. This
control loop can therefore be seen as a State Machine architecture, with 4 parallel systems (cavities)
being controlled. In this state machine, each system can be acted on by one of 4 states:
1) Safe: Output 0 voltages and falsify all booleans. Basically, do nothing.
2) Hold: Pause the output voltage and hold this constant until further notice.
3) Scan: Apply a variable saw-tooth wave voltage in order to scan the cavity’s optical length.
4) Lock: Apply a PII controller on the error signal and lock the cavity on resonance.
