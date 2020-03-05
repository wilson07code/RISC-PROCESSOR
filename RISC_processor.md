# RISC_PROCESSOR
------top-----

module RISC_top(
input clk


    );

    reg [26:0]q;
    reg clk_div;


  wire cin,rst;

  wire [3:0]opcode;

   wire [3:0]oprand_1;

  wire  [7:0]oprand_2;

  wire    [15:0]alu_output;

  wire  cb_r ;



    wire [15:0]din;
    wire [15:0]dout;
    wire [3:0]addr_r,operation_r;
    wire [15:0]alu_opr1_r;
    wire  [7:0]alu_opr2_r;
    wire cs_r,wb_r,r_wbar_r;


    always@(posedge clk)
    begin
    if(rst)
    begin
    clk_div<=1'b0;
    q<=1'b0;
    end
    else
    begin
    q<=q+1;
    if(q<=100000000/2)
    begin
    clk_div<=~clk_div;
    q<=0;
    end
   end
  end

    instruction_decoder
id(.rst(rst),.clk(clk_div),.opcode(opcode),.opr1(oprand_1),.opr2(oprand_2),

.dout(dout),.wb(wb_r),.cs(cs_r),.r_wbar(r_wbar_r),.addr(addr_r),

.alu_operation(operation_r),.alu_opr1(alu_opr1_r),.alu_opr2(alu_opr2_r));

     alu a(.operation(operation_r),.clk(clk_div),.rst(rst),.cin(cin),.opr1(alu_opr1_r),
            .opr2(alu_opr2_r),.alu_out(alu_output),.cb(cb_r));

     memory mem(.clk(clk_div),.rst(rst),.cs(cs_r),.r_wbar(r_wbar_r),.addr(addr_r),.din(din),.dout(dout));

     mux mx(.wb(wb_r),.dout(dout),.alu_out(alu_output),.din(din));


     vio_0 your_instance_name (
  .clk(clk),                // input wire clk
  .probe_in0(alu_output),    // input wire [15 : 0] probe_in0
  .probe_in1(cb_r),    // input wire [0 : 0] probe_in1
  .probe_out0(opcode),  // output wire [3 : 0] probe_out0
  .probe_out1(oprand_1),  // output wire [3 : 0] probe_out1
  .probe_out2(oprand_2),  // output wire [7 : 0] probe_out2
  .probe_out3(rst),  // output wire [0 : 0] probe_out3
  .probe_out4(cin)  // output wire [0 : 0] probe_out4
);

endmodule


----------alu----------------


module alu(
input [3:0]operation,
input clk,rst,cin,
input [15:0]opr1,
input [7:0]opr2,
output reg [15:0]alu_out,
output reg cb  );

wire [15:0]sn_ex;

assign sn_ex={{8{opr2[7]}},opr2};


always@(posedge clk)
begin


if(rst)
begin
alu_out<=16'h0;
cb<=1'b0;
end

else
case(operation)
4'b0000:begin
         if(cin)
         {cb,alu_out}<=opr1+sn_ex+cin;
         else
         {cb,alu_out}<=opr1+sn_ex;
         end
4'b0001:begin
        if(cin)
        {cb,alu_out}<=opr1-sn_ex;
        else
        {cb,alu_out}<=opr1-sn_ex-1'b1;
        end
 4'b0010: {cb,alu_out}<=opr1+1'b1;

 4'b0011: {cb,alu_out}<=opr1-1'b1;

 4'b0100: {cb,alu_out}<=opr1&sn_ex;

 4'b0101: {cb,alu_out}<=opr1|sn_ex;

 4'b0110: {cb,alu_out}<=~opr1;

 4'b0111: {cb,alu_out}<=~(opr1&sn_ex);

 4'b1000: {cb,alu_out}<=~(opr1|sn_ex);

 4'b1001: {cb,alu_out}<=opr1 ^ sn_ex;

  4'b1010: {cb,alu_out}<=~(opr1 ^ sn_ex);

   4'b1011: {cb,alu_out}<=opr1<<1;

   4'b1100: {cb,alu_out}<=opr1>>1;

   4'b1101 : {cb,alu_out}<={opr1[14:0],opr1[0]};

   4'b1110 : {cb,alu_out}<={opr1[15],opr1[15:1]};

   default : {cb,alu_out}<=16'h0;



endcase
end
endmodule

----------memory------------


module memory(
input clk,rst,cs,r_wbar,
input [3:0]addr,
input [15:0]din,output reg [15:0]dout
    );
    reg [15:0]mem[0:15];

    always@(posedge clk)
    begin
    if(rst)
    begin
   mem[4'b0000] =16'h01;
    mem[4'b0001 ]=16'h02;
    mem[4'b0010] =16'h03;
    mem[4'b0011] =16'h04;
    mem[4'b0100] =16'h05;
    mem[4'b0101] =16'h06;
    mem[4'b0110] =16'h32;
    mem[4'b0111] =16'h50;
    mem[4'b1000] =16'h30;
    mem[4'b1001] =16'h25;
    mem[4'b1010] =16'h40;
    mem[4'b1011] =16'h61;
    mem[4'b1100] =16'h72;
    mem[4'b1101] =16'h83;
    mem[4'b1110] =16'h94;
    mem[4'b1111]=16'h105;
  end
  else
  if(cs)
  begin
    if(r_wbar==1'b0)
        mem[addr]<=din;
     else
        dout<=mem[addr];
    end
  else
        dout<=dout;
 end

endmodule


-----------mux-----------


module mux(
input wb,
input [15:0]dout,alu_out,
output reg [15:0]din
    );

 always@(wb,dout,alu_out)
 begin
    if(wb)
    din<=alu_out;
    else
    din<=dout;
  end
endmodule


---------I.decoder--------


module instruction_decoder(
input rst,clk,
input [3:0]opcode,[3:0]opr1,[7:0]opr2,
input [15:0]dout,
output reg wb,cs,r_wbar,
output reg [3:0]addr,alu_operation,
output reg [15:0]alu_opr1,
output reg [7:0]alu_opr2
    );
    reg [2:0]state,next;

  parameter INIT=3'd0,
            FETCH=3'd1,
            DECODE=3'd2,
            EXECUTE=3'd3,
            LOAD=3'd4;


    always@(posedge clk,posedge rst)
    begin
    if(rst)
    state<=INIT;
    else
    state<=next;
    end

    always@(rst,state,opcode,opr1,opr2)
    begin

    case(state)

    INIT: if(rst)
         next<=INIT;
         else
         next<=FETCH;


    FETCH: begin
            alu_operation<=opcode;
           addr<=opr1;
           alu_opr2<=opr2;
           r_wbar<=1'b1;
           cs<=1'b1;
           wb<=1'b0;
           next<=DECODE;
           end


     DECODE:begin
              alu_opr1<=dout;
             wb<=1'b0;
             cs<=1'b0;
             r_wbar<=1'b0;
             next<=EXECUTE;
            end

     EXECUTE: begin
             alu_opr1<=dout;
             wb<=1'b1;
             cs<=1'b1;
             r_wbar<=1'b0;
             next<=LOAD;
              end

      LOAD: begin
            r_wbar<=1'b0;
            wb<=1'b1;
            cs<=1'b0;
            next<=FETCH;
            end

      endcase

    end

endmodule
