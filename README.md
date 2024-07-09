# vlsi-PROJECT-2-Memory-controller-design-2
This is a very a simple sdram controller which works on the De0 Nano. The project also contains a simple push button interface for testing on the dev board.

Basic features

Operates at 100Mhz, CAS 3, 32MB, 16-bit data
On reset will go into INIT sequnce
After INIT the controller sits in IDLE waiting for REFRESH, READ or WRITE
REFRESH operations are spaced evenly 8192 times every 32ms
READ is always single read with auto precharge
WRITE is always single write with auto precharge
 
 Host Interface          SDRAM Interface

   /-----------------------------\
   |      sdram_controller       |
==> wr_addr                  addr ==>
==> wr_data             bank_addr ==>
--> wr_enable                data <=>
   |                 clock_enable -->
==> rd_addr                  cs_n -->   
--> rd_enable               ras_n -->
<== rd_data                 cas_n -->
<-- rd_ready                 we_n -->
<-- busy            data_mask_low -->      
   |               data_mask_high -->
--> rst_n                        |
--> clk                          |
   \-----------------------------/

From the above diagram most signals should be pretty much self explainatory. Here are some important points for now. It will be expanded on later.

wr_addr and rd_addr are equivelant to the concatenation of {bank, row, column}
rd_enable should be set to high once an address is presented on the addr bus and we wish to read data.
wr_enable should be set to high once addr and data is presented on the bus
busy will go high when the read or write command is acknowledged. busy will go low when the write or read operation is complete.
rd_ready will go high when data rd_data is available on the data bus.
NOTE For single reads and writes wr_enable and rd_enable should be set low once busy is observed. This will protect from the controller thinking another request is needed if left higher any longer.
Build
The recommended way to build is to use fusesoc. The build steps are then:
# Build the project with quartus
fusesoc build dram_controller
# Program the project to de0 nano
fusesoc pgm dram_controller

# Build with icarus verilog and test
fusesoc sim dram_controller --vcd
gtkwave $fusebuild/dram_controller/sim-icarus/testlog.vcd

# Run other test cases 
fusesoc sim --testbench fifo_tb dram_controller --vcd
fusesoc sim --testbench double_click_tb dram_controller --vcd
