module vlog_tb_utils;
   parameter MAX_STRING_LEN = 128;
   localparam CHAR_WIDTH = 8;

   //Force simulation stop after timeout cycles
   reg [63:0] timeout;
   initial
     if($value$plusargs("timeout=%d", timeout)) begin
	#timeout $display("Timeout: Forcing end of simulation");
	$finish;
     end

   //FIXME: Add more options for VCD logging
   reg [MAX_STRING_LEN*CHAR_WIDTH-1:0] testcase;

   initial begin
      if($test$plusargs("vcd")) begin
	 if($value$plusargs("testcase=%s", testcase))
	   $dumpfile({testcase,".vcd"});
	 else
	   $dumpfile("testlog.vcd");
	 $dumpvars(0);
      end
   end

   //Heartbeat timer for simulations
   reg [63:0] heartbeat;
   initial begin
      if($value$plusargs("heartbeat=%d", heartbeat))
	forever #heartbeat $display("Heartbeat : Time=%0t", $time); 
   end
///////////////////////////////////////////////////////////////////////////////////////////////////////
// test if all parameters are set
reg [63:0] FItime;  // fault injection time 
reg [63:0] logTime;  // fault injection time 
reg [6:0]  FIbit;
reg [4:0] FIcomponent;
integer    fiLog = 0;
reg faultIsInjected=0;
reg [31:0] ALUFI=32'b0;
//DECODER FI signals
reg [31:0] DECODERFIONDECODEDINST=32'b0;
reg [31:0] DECODERFIONDECODEDPC=32'b0;
reg FI_on_fetch_valid_o=1'b0;
reg [31:0] FI_on_ibus_dat=32'b0;
reg [31:0] FI_on_pc_fetch=32'b0;
reg [31:0] FI_on_pc_addr=32'b0;
reg [31:0] FI_on_ibus_adr=32'b0;
reg FI_on_ibus_req=1'b0;
reg FI_on_ibus_ack=1'b0;
reg [2:0] FI_on_state=3'b0;
reg FI_on_ctrl_branch_exception_r=1'b0;
reg  FI_on_decode_except_ibus_err_o=1'b0;
reg   FI_on_flush=1'b0;
reg 	FI_on_fetching_brcond=1'b0;
reg 	FI_on_fetching_mispredicted_branch=1'b0;
//DECODE EXECUTE FAULT INJECTION
reg [31:0] FI_on_decode_execute_mispredict_target=32'b0;
reg [31:0] FI_on_decode_execute_execute_immediate_o=32'b0;
reg [31:0] FI_on_decode_execute_jal_result_o=32'b0;
reg [31:0] FI_on_decode_execute_pc_execute_o=32'b0;
reg FI_on_decode_execute_bubble_o=1'b0;
reg FI_on_decode_execute_valid_o=1'b0;
reg [31:0] FI_on_decode_execute_ctrl=32'b0; // on ctrl bits
reg [6:0] FI_on_decode_execute_except=7'b0; // on ctrl bits
reg [5:0] FI_on_decode_execute_opc_insn_o=6'b0; // on ctrl bits
reg [4:0] FI_on_decode_execute_rfd_adr_o=5'b0;
reg [9:0] FI_on_decode_execute_immjbr_upper_o=10'b0;
reg FI_on_decode_execute_adder_do_sub_o=1'b0;
reg FI_on_decode_execute_immediate_sel_o=1'b0;
reg[15:0] FI_on_decode_execute_imm16_o=16'b0;
reg[3:0] FI_on_decode_execute_opc_alu_o=4'b0;
reg[3:0] FI_on_decode_execute_opc_alu_secondary_o=4'b0;

////fault injection on mux_WB stage
reg[32:0] FI_on_rf_result=33'b0;
////fault injection on execute_ctrl_cappacuiono 182 bits
reg[32:0]  FI_on_ctrl_alu_result_o=32'b0;
reg[32:0]  FI_on_ctrl_lsu_adr_o=32'b0;
reg[32:0]  FI_on_ctrl_rfb_o=32'b0;
reg[32:0]  FI_on_pc_ctrl_o=32'b0;
reg [4:0] FI_on_ctrl_rfd_adr_o=5'b0;
reg [4:0] FI_on_wb_rfd_adr_o=5'b0;
reg FI_on_ctrl_rf_wb_o=1'b0;
reg FI_on_wb_rf_wb_o=1'b0;
reg[12:0] FI_on_ctrl_fpcsr_o=13'b0;
reg[27:0] FI_on_execute_ctrl_ctrl_bits=28'b0;

///fault injection on ctrl_cappacciono
reg[10:0]  FI_on_ctrl=11'b0; //ctrl_bubble_o, execute_delay_slot, ctrl_delay_slot, execute_waiting_r,decode_execute_halt,exception_taken, padv_ctrl, exception_r,waiting_for_fetch, doing_rfe_r
//ctrl_stage_exceptions, 
reg[31:0]  FI_on_spr_epcr=32'b0;
reg[31:0]  FI_on_spr_ppc=32'b0;
reg[31:0]  FI_on_spr_npc=32'b0;
reg[31:0]  FI_on_last_branch_insn_pc=32'b0;
reg[31:0]  FI_on_last_branch_target_pc=32'b0;
reg[31:0]  FI_on_exception_pc_addr=32'b0;
reg[31:0]  FI_on_spr_sys_group_read=32'b0;
//////////////////






///

initial  begin
if (!$value$plusargs("FItime=%d", FItime)) begin
	$display("*************Golden RUN***************");
	end
else if (!$value$plusargs("FIcomponent=%d", FIcomponent)) begin
	$display("*************Error: Please indentify the fault injection component (--FIcomponent=1/2/3/4)***************");
	$finish;
	end
else if  (!$value$plusargs("FIbit=%d", FIbit)) begin
	$display("*************Error: Please indentify the fault injection bit (--FIbit=0~31)***************");
	$finish;
	end
        fiLog = $fopen("faultinjection.log");
end

`include "/home/abudhira/or1k/mor1kxRTL/mor1kx/rtl/verilog/FaultInjection/FIOnALU.v"
`include "/home/abudhira/or1k/mor1kxRTL/mor1kx/rtl/verilog/FaultInjection/FIOnDecode.v"
`include "/home/abudhira/or1k/mor1kxRTL/mor1kx/rtl/verilog/FaultInjection/FIOnlsqData.v"
`include "/home/abudhira/or1k/mor1kxRTL/mor1kx/rtl/verilog/FaultInjection/FIOnlsqAddress.v"
`include "/home/abudhira/or1k/mor1kxRTL/mor1kx/rtl/verilog/FaultInjection/FIOnRegFile.v"
`include "/home/abudhira/or1k/mor1kxRTL/mor1kx/rtl/verilog/FaultInjection/FIOnDecodeExecute.v"
`include "/home/abudhira/or1k/mor1kxRTL/mor1kx/rtl/verilog/FaultInjection/FIOnExecuteCtrl.v"
`include "/home/abudhira/or1k/mor1kxRTL/mor1kx/rtl/verilog/FaultInjection/FIOnCtrl.v"
`include "/home/abudhira/or1k/mor1kxRTL/mor1kx/rtl/verilog/FaultInjection/FIOnHazardDetectors.v"
`include "/home/abudhira/or1k/mor1kxRTL/mor1kx/rtl/verilog/FaultInjection/FIOnWbMux.v"


// STEP2: Fault Injection
   initial begin
   if (FIcomponent == 1) faultInjectionOnALUResults(); // FIcomponent = 1 means fault injection on ALU results (this is actually registers in alu_ctrl) 32 bits
   if (FIcomponent == 2) faultInjectionOnFetchtoDecodeRegs(); // FIcomponent = 2means fault injection on all registers in fetch and decode units Aound 198 bits
   if (FIcomponent == 3) faultInjectionOnlsqData(); // FIcomponent = 3 means fault injection on data registers of LSQ
   if (FIcomponent == 4) faultInjectionOnlsqAddress(); // FIcomponent = 4 means fault injection on address registers of LSQ
   if (FIcomponent == 5) faultInjectionOnRegFile(); // FIcomponent = 5 means fault injection on register file Aound 198 bits (32*32) 1064 bits
   if (FIcomponent == 6) faultInjectionOnDecodetoExecuteRegs(); // FIcomponent = 6 means fault injection on decode_execute reggisters: 216 bits
   if (FIcomponent == 7) faultInjectionOnExecuteCtrl(); // FIcomponent = 7 means fault injection on execute_ctrl_cappacinou: 182 BITS
   if (FIcomponent == 8) faultInjectionHazardDetectors(); // FIcomponent = 8 means fault injection on all bits inside rf_control units (hazard detection unit) Aound 232 bits
   if (FIcomponent == 9) faultInjectionWbMux(); // FIcomponent = 8 means fault injection on all bits inside write back (to register file or memory) stage of the pipeline Aound 33 bits
   if (FIcomponent == 10) faultInjectionCtrl(); // FIcomponent = 10 means fault injection on all bits mor1kx_ctrl_cappuccion.v 235 Bits
   end


endmodule // vlog_tb_utils

//#5000 release orpsoc_tb.dut.mor1kx0.mor1kx_cpu.cappuccino.mor1kx_cpu.mor1kx_execute_alu.alu_result_o[31:0];

