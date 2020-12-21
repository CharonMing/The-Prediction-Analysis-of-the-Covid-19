# The Prediction analysis of covid-19
module lift_study(
  input clk_100MHz,
  input [4:0] s,
  output [4:0] nfloor,
  output [10:0] seg,
  output reg lift_open//电梯是否开门，1为开，0为关
    );
  parameter N=5;    
  reg clk_1s;//电梯速度
  reg clk_200ms;//电梯开门时间 
  reg [4:0] btn_pre_re,btn_buff,btn_off,btn_test;//按键记录变量
  reg [25:0] count,count1,count3;
  reg [10:0] dout;
  reg [1:0] lift_state;//电梯状态
  reg [2:0] lift_num;//电梯位置
  initial begin
    btn_off<=5'b11111;
    btn_pre_re<=0;
    lift_num <= 3;
    lift_state<=0;
    lift_open<=1;
    clk_200ms <=0;
    clk_1s <=0;  
  end
  //    按键防防抖时间设置
  always@(posedge clk_100MHz)
    begin
      count <= count + 1;
      if(count==24'h98967F)//4C4B3F)
        begin
          count <= 0;
          clk_200ms <= ~clk_200ms;
    end
  end             
  // 电梯速度 
  always@(posedge clk_100MHz)
    begin
      count1 <= count1 + 1; 
      if(count1 ==26'h2FAF07F)
        begin
          count1 <= 0;
          clk_1s <= ~clk_1s;
        end
    end      
//    记录按键信息 
  always @(posedge clk_200ms) 
    begin
      btn_pre_re=btn_pre_re^s;
      btn_pre_re=btn_pre_re&btn_off; 
    end       
  always@(posedge clk_1s) 
    begin
      btn_buff=btn_pre_re;
      case(lift_state)
        0:begin
          if((btn_buff>>lift_num)>0) 
            begin 
              #(5*N);
              lift_num=lift_num+1;
              lift_state=1;//上层有人叫梯
            end                   
          if((btn_buff&(1<<(lift_num-1)))>0) //本层有人叫梯
            begin
              btn_buff=btn_buff&(~(1<<(lift_num-1)));
              btn_off=~(1<<(lift_num-1));
              lift_open=1;
              #(5*N);
              lift_open=0;
              lift_state=0;
            end
          if((1<<(lift_num-1))>btn_buff) //下层有人叫梯
            begin
              if(btn_buff>0) 
                begin
                  #(5*N);
                  lift_num=lift_num-1;
                  lift_state=2;
                end
            end
          end
        1:begin
          if((btn_buff>>lift_num)>0)
            begin
              if((btn_buff&(1<<(lift_num-1)))>0) 
                begin
                  btn_buff=btn_buff&(~(1<<(lift_num-1)));
                  btn_off=~(1<<(lift_num-1));                        
                  lift_open=1;
                  #(5*N);
                  lift_open=0;  
                  lift_num=lift_num+1;
                end
              else
                  lift_num=lift_num+1;    
            end
          else 
            begin
              btn_buff=btn_buff&(~(1<<(lift_num-1)));
              btn_off=~(1<<(lift_num-1));
              lift_open=1;
              #(5*N);
              lift_open=0;
              lift_state=0;
            end    
          end
        2: begin
          btn_test=(btn_buff<<(6-lift_num));
          if(btn_test>0) 
            begin
              if((btn_buff&(1<<(lift_num-1)))>0) 
                begin
                  btn_buff=btn_buff&(~(1<<(lift_num-1)));
                  btn_off=~(1<<(lift_num-1));
                  lift_open=1;
                  #(5*N);
                  lift_open=0;
                  lift_num=lift_num-1;                        
                end
              else
                lift_num=lift_num-1;
            end
          else 
            begin
              btn_buff=btn_buff&(~(1<<(lift_num-1)));
              btn_off=~(1<<(lift_num-1));
              lift_open=1;
              #(5*N);
              lift_open=0;               
              lift_state=0;
            end           
          end                               
        default :lift_state<=0;
      endcase
    end
  always@(posedge clk_100MHz) 
    begin
      case(lift_num)
        0:dout = 11'b0001_1111110;
        1:dout = 11'b0001_0110000;
        2:dout = 11'b0001_1101101;
        3:dout = 11'b0001_1111001;
        4:dout = 11'b0001_0110011;
        5:dout = 11'b0001_1011011;
        6:dout = 11'b0001_1010111;
        7:dout = 11'b0001_1110000;
        8:dout = 11'b0001_1111111;
        9:dout = 11'b0001_1111011;
        default:dout = 11'b0001_1111110;     
      endcase
    end
  assign nfloor=btn_pre_re;
  assign seg=dout;        
endmodule
（5）约束文件
set_property -dict {PACKAGE_PIN U4 IOSTANDARD LVCMOS33} [get_ports {s[4]}]
set_property -dict {PACKAGE_PIN V1 IOSTANDARD LVCMOS33} [get_ports {s[3]}]
set_property -dict {PACKAGE_PIN R15 IOSTANDARD LVCMOS33} [get_ports {s[2]}]
set_property -dict {PACKAGE_PIN R17 IOSTANDARD LVCMOS33} [get_ports {s[1]}]
set_property -dict {PACKAGE_PIN R11 IOSTANDARD LVCMOS33} [get_ports {s[0]}]
set_property -dict {PACKAGE_PIN G6 IOSTANDARD LVCMOS33} [get_ports {seg[10]}]
set_property -dict {PACKAGE_PIN E1 IOSTANDARD LVCMOS33} [get_ports {seg[9]}]
set_property -dict {PACKAGE_PIN F1 IOSTANDARD LVCMOS33} [get_ports {seg[8]}]
set_property -dict {PACKAGE_PIN G1 IOSTANDARD LVCMOS33} [get_ports {seg[7]}]
set_property -dict {PACKAGE_PIN D4 IOSTANDARD LVCMOS33} [get_ports {seg[6]}]
set_property -dict {PACKAGE_PIN E3 IOSTANDARD LVCMOS33} [get_ports {seg[5]}]
set_property -dict {PACKAGE_PIN D3 IOSTANDARD LVCMOS33} [get_ports {seg[4]}]
set_property -dict {PACKAGE_PIN F4 IOSTANDARD LVCMOS33} [get_ports {seg[3]}]
set_property -dict {PACKAGE_PIN F3 IOSTANDARD LVCMOS33} [get_ports {seg[2]}]
set_property -dict {PACKAGE_PIN E2 IOSTANDARD LVCMOS33} [get_ports {seg[1]}]
set_property -dict {PACKAGE_PIN D2 IOSTANDARD LVCMOS33} [get_ports {seg[0]}]
set_property -dict {PACKAGE_PIN F6 IOSTANDARD LVCMOS33}[get_ports lift_open]
set_property -dict {PACKAGE_PIN P17 IOSTANDARD LVCMOS33} [get_ports {clk_100MHz}]
set_property -dict {PACKAGE_PIN J4 IOSTANDARD LVCMOS33} [get_ports {nfloor[4]}]
set_property -dict {PACKAGE_PIN H4 IOSTANDARD LVCMOS33} [get_ports {nfloor[3]}]
set_property -dict {PACKAGE_PIN J3 IOSTANDARD LVCMOS33} [get_ports {nfloor[2]}]
set_property -dict {PACKAGE_PIN J2 IOSTANDARD LVCMOS33} [get_ports {nfloor[1]}]
set_property -dict {PACKAGE_PIN K2 IOSTANDARD LVCMOS33} [get_ports {nfloor[0]}]
