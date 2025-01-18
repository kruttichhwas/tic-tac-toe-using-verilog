`timescale 1ns / 1ps
module tictactoe(
    input clk,
    input [3:0]position,                // 0000 to 1001 position value of box to mark
    input load,                         // 1 - loads current player's data
    output reg [6:0]display,        // for controlling ssd display, bits are [gfedcba] 
    output reg [2:0]move,      // red and green -player 1 and blue-player 2 [bgr]
    output reg [3:0]ssd                // for selecting ssd
    );
//-------------------------------------------------------------------inside module variable declarations     
    reg [1:0]current_player;
    reg [8:0]playerone;                // moves of player one
    reg [8:0]playertwo;                // moves of player two
    reg [14:0]counter_400hz;
    reg [8:0]clk_400hz;
    reg [1:0]winner;
    integer i;                        // used in for loop operations
//----------------------------------------------------------------initial value assignments     
    initial begin
        current_player = 2'b01;
        playerone = 9'b000000000;
        playertwo = 9'b000000000;
        counter_400hz = 15'b000000000000000;
        clk_400hz = 9'b000000000;
        winner = 2'b00;
    end
//----------------------------------------------------------------analysing player movement  
    always@(*)
    begin
        if((load == 1'b1) && (playerone[position] == 1'b0) && (playertwo[position] == 1'b0) && (position <= 4'b1001) && (winner == 2'b00))
        begin
            if(current_player==2'b01)
            begin
               playerone[position] = 1'b1;
               current_player = 2'b10;
               move = 3'b010; 
            end
            else
            begin
               playertwo[position] = 1'b1;
               current_player = 2'b01;
               move = 3'b100;     
            end
        end
        else
        begin
            move = 3'b001;
        end
    end
//--------------------------making 400hz clock for ssd display updation, each ssd - 100hz     
    always@(posedge clk) 
    begin
        if(counter_400hz == 15'b111010100110000)
        begin
            counter_400hz = 15'b000000000000000;
            if(clk_400hz == 9'b110010000)
            begin
                clk_400hz = 9'b000000000;
            end
            else
            begin
                clk_400hz = clk_400hz + 1;
            end
        end
        else
        begin
            counter_400hz = counter_400hz + 1;
        end 
    end
//------------------------------------------------------------------updating ss displays    
    always@(posedge clk_400hz)
    begin
        if(winner == 2'b00)
        begin
            case(clk_400hz[1:0])
                2'b00:  // update ssd 1 and turn all other off , rightmost
                    begin
                        ssd = 4'b0001;
                        if(current_player == 2'b01)
                        begin
                            display = 7'b0000110;      // 1  
                        end
                        else
                        begin
                            display = 7'b1011011;      // 2
                        end 
                    end
                2'b01:
                    begin
                        ssd = 4'b0010;
                        display = 7'b0000000;
                        for(i = 2; i <= 8; i = i + 3)
                        begin
                            if(playerone[i] == 1'b1)
                            begin
                                display[i] = 1'b1;
                            end
                            else if((playertwo[i] == 1'b1) && (clk_400hz[8] == 1'b1))
                            begin
                                display[i] = 1'b1; 
                            end
                        end
                    end
                2'b10:
                    begin
                        ssd = 4'b0100;
                        display = 7'b0000000;
                        for(i = 1; i <= 7; i = i + 3)
                        begin
                            if(playerone[i] == 1'b1)
                            begin
                                display[i] = 1'b1;
                            end
                            else if((playertwo[i] == 1'b1) && (clk_400hz[8] == 1'b1))
                            begin
                                display[i] = 1'b1; 
                            end
                        end
                    end            
                2'b11:
                    begin
                        ssd = 4'b1000;
                        display = 7'b0000000;
                        for(i = 0; i <= 6; i = i + 3)
                        begin
                            if(playerone[i] == 1'b1)
                            begin
                                display[i] = 1'b1;
                            end
                            else if((playertwo[i] == 1'b1) && (clk_400hz[8] == 1'b1))
                            begin
                                display[i] = 1'b1; 
                            end
                        end
                    end            
            endcase
        end
    end
//-----------------------------------------------------------check for winning pattern
    always@(*)
    begin
        if((playerone == 9'b111000000) || (playerone == 9'b000111000) || (playerone == 9'b000000111) || (playerone == 9'b100100100) || (playerone == 9'b010010010) || (playerone == 9'b001001001) || (playerone == 9'b001010100) || (playerone == 9'b100010001))
        begin
            winner = 2'b01;
            ssd = 4'b1111;
            display = 7'b0000110;
        end
        else if((playertwo == 9'b111000000) || (playertwo == 9'b000111000) || (playertwo == 9'b000000111) || (playertwo == 9'b100100100) || (playertwo == 9'b010010010) || (playertwo == 9'b001001001) || (playertwo == 9'b001010100) || (playertwo == 9'b100010001))
        begin
            winner = 2'b01;
            ssd = 4'b1111;
            display = 7'b1011011;
        end
    end
//----------------------------------------------------------------------------------end     
endmodule