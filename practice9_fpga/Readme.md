# LogicDesign

논리회로실험 2019.11.26(화)

## Practice09
### IR 통신방법
- 송신부 : 발광 다이오드
- 수신부 : 포토 다이오드

### 코드
1) nco
```verilog 
module	nco(	
		o_gen_clk,
		i_nco_num,
		clk,
		rst_n);

output		o_gen_clk	;	// 1Hz CLK

input	[31:0]	i_nco_num	;
input		clk		;	// 50Mhz CLK
input		rst_n		;

reg	[31:0]	cnt		;
reg		o_gen_clk	;

always @(posedge clk or negedge rst_n) begin
	if(rst_n == 1'b0) begin
		cnt		<= 32'd0;
		o_gen_clk	<= 1'd0	;
	end else begin
		if(cnt >= i_nco_num/2-1) begin
			cnt 	<= 32'd0;
			o_gen_clk	<= ~o_gen_clk;
		end else begin
			cnt <= cnt + 1'b1;
		end
	end
end
endmodule
```  
clk의 Hz를 50MHz에서 1Hz로 바꿔줌

2) fnd_dec
```verilog 
module	fnd_dec(
		o_seg,
		i_num);

output	[6:0]	o_seg		;	// {o_seg_a, o_seg_b, ... , o_seg_g}

input	[3:0]	i_num		;
reg	[6:0]	o_seg		;
//making
always @(i_num) begin 
 	case(i_num) 
 		4'd0:	o_seg = 7'b111_1110; 
 		4'd1:	o_seg = 7'b011_0000; 
 		4'd2:	o_seg = 7'b110_1101; 
 		4'd3:	o_seg = 7'b111_1001; 
 		4'd4:	o_seg = 7'b011_0011; 
 		4'd5:	o_seg = 7'b101_1011; 
 		4'd6:	o_seg = 7'b101_1111; 
 		4'd7:	o_seg = 7'b111_0000; 
 		4'd8:	o_seg = 7'b111_1111; 
 		4'd9:	o_seg = 7'b111_0011; 
 		4'd10:	o_seg = 7'b111_0111; 
 		4'd11:	o_seg = 7'b001_1111; 
 		4'd12:	o_seg = 7'b100_1110; 
 		4'd13:	o_seg = 7'b011_1101; 
 		4'd14:	o_seg = 7'b100_1111; 
 		4'd15:	o_seg = 7'b100_0111; 
		default:o_seg = 7'b000_0000; 
	endcase 
end
endmodule
```
각 숫자(1-9)와 알파벳(A-F)을 7-세그먼트로 나타냄

3) double_fig_sep
```verilog 
module	double_fig_sep(
		o_left,
		o_right,
		i_double_fig);

output	[3:0]	o_left		;
output	[3:0]	o_right		;

input	[5:0]	i_double_fig	;

assign		o_left	= i_double_fig / 10	;
assign		o_right	= i_double_fig % 10	;

endmodule
```
4bit의 숫자가 10진법으로 10의 자리 수와 1의 자리 수로 o_left와 o_right에 값을 할당함

4) led_disp
```verilog 
module	led_disp(
		o_seg,
		o_seg_dp,
		o_seg_enb,
		i_six_digit_seg,
		i_six_dp,
		clk,
		rst_n);

output	[5:0]	o_seg_enb		;
output		o_seg_dp		;
output	[6:0]	o_seg			;

input	[41:0]	i_six_digit_seg		;
input	[5:0]	i_six_dp		;
input		clk			;
input		rst_n			;

wire		gen_clk		;

nco		u_nco(
		.o_gen_clk	( gen_clk	),
		.i_nco_num	( 32'd5000	),
		.clk		( clk		),
		.rst_n		( rst_n		));


reg	[3:0]	cnt_common_node	;

always @(posedge gen_clk or negedge rst_n) begin
	if(rst_n == 1'b0) begin
		cnt_common_node <= 4'd0;
	end else begin
		if(cnt_common_node >= 4'd5) begin
			cnt_common_node <= 4'd0;
		end else begin
			cnt_common_node <= cnt_common_node + 1'b1;
		end
	end
end

reg	[5:0]	o_seg_enb		;

always @(cnt_common_node) begin
	case (cnt_common_node)
		4'd0:	o_seg_enb = 6'b111110;
		4'd1:	o_seg_enb = 6'b111101;
		4'd2:	o_seg_enb = 6'b111011;
		4'd3:	o_seg_enb = 6'b110111;
		4'd4:	o_seg_enb = 6'b101111;
		4'd5:	o_seg_enb = 6'b011111;
		default:o_seg_enb = 6'b111111;
	endcase
end

reg		o_seg_dp		;

always @(cnt_common_node) begin
	case (cnt_common_node)
		4'd0:	o_seg_dp = i_six_dp[0];
		4'd1:	o_seg_dp = i_six_dp[1];
		4'd2:	o_seg_dp = i_six_dp[2];
		4'd3:	o_seg_dp = i_six_dp[3];
		4'd4:	o_seg_dp = i_six_dp[4];
		4'd5:	o_seg_dp = i_six_dp[5];
		default:o_seg_dp = 1'b0;
	endcase
end

reg	[6:0]	o_seg			;

always @(cnt_common_node) begin
	case (cnt_common_node)
		4'd0:	o_seg = i_six_digit_seg[6:0];
		4'd1:	o_seg = i_six_digit_seg[13:7];
		4'd2:	o_seg = i_six_digit_seg[20:14];
		4'd3:	o_seg = i_six_digit_seg[27:21];
		4'd4:	o_seg = i_six_digit_seg[34:28];
		4'd5:	o_seg = i_six_digit_seg[41:35];
		default:o_seg = 7'b111_1110; // 0 display
	endcase
end

endmodule
```

## ir_rx
``` verilog
//		Sequential Rx Bits

wire		ir_rx		;
assign		ir_rx = ~i_ir_rxb;

reg	[1:0]	seq_rx				;
always @(posedge clk_1M or negedge rst_n) begin
	if(rst_n == 1'b0) begin
		seq_rx <= 2'b00;
	end else begin
		seq_rx <= {seq_rx[0], ir_rx};
	end
end
```
seq_rx에서 이전값과 현재값을 저장하며 연속적인 수신을 받는다.
seq_rx[0] 이전값
ir_rx 현재값

``` verilog
//		Count Signal Polarity (High & Low)
reg	[15:0]	cnt_h		;
reg	[15:0]	cnt_l		;
always @(posedge clk_1M or negedge rst_n) begin
	if(rst_n == 1'b0) begin
		cnt_h <= 16'd0;
		cnt_l <= 16'd0;
	end else begin
		case(seq_rx)
			2'b00	: cnt_l <= cnt_l + 1;
			2'b01	: begin
				cnt_l <= 16'd0;
				cnt_h <= 16'd0;
			end
			2'b11	: cnt_h <= cnt_h + 1;
		endcase
	end
end
```
- state를 parameter 네 개로 나눠 정의.
- high/low 로 나눠서 확인함.
- 2'b01로 다시 리셋시킴.

``` verilog
//		State Machine
reg	[1:0]	state		;
reg	[5:0]	cnt32		;
always @(posedge clk_1M or negedge rst_n) begin
	if(rst_n == 1'b0) begin
		state <= IDLE;
		cnt32 <= 6'd0;
	end else begin
		case (state)
			IDLE: begin
				state <= LEADCODE;
				cnt32 <= 6'd0;
			end
			LEADCODE: begin
				if (cnt_h >= 8500 && cnt_l >= 4000) begin
					state <= DATACODE;
				end else begin
					state <= LEADCODE;
				end
			end
			DATACODE: begin
				if (seq_rx == 2'b01) begin
					cnt32 <= cnt32 + 1;
				end else begin
					cnt32 <= cnt32;
				end
				if (cnt32 >= 6'd32 && cnt_l >= 1000) begin
					state <= COMPLETE;
				end else begin
					state <= DATACODE;
				end
			end
			COMPLETE: state <= IDLE;
		endcase
	end
end
``` 
- seq_rx는 rising edge

``` verilog
//		32bit Custom & Data Code
reg	[31:0]	data		;
reg	[31:0]	o_data		;
always @(posedge clk_1M or negedge rst_n) begin
	if(rst_n == 1'b0) begin
		data <= 32'd0;
	end else begin
		case (state)
			DATACODE: begin
				if (cnt_l >= 1000) begin
					data[32-cnt32] <= 1'b1;
				end else begin
					data[32-cnt32] <= 1'b0;
				end
			end
			COMPLETE: o_data <= data;
		endcase
	end
end


endmodule
``` 
- 32 bit data가 신호를 만든다. 

### waveform  사진
![](https://github.com/hyeyeonkang/LogicDesign/blob/master/practice9_fpga/prj09.PNG)


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwOTgyNjA3NjcsLTExNDE4NTM0NjNdfQ
==
-->