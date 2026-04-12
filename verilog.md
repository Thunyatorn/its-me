module hello_output (
    input  wire        clk,
    input  wire        rst_n,
    input  wire        en,
    output reg  [7:0]  char_out,
    output reg         char_valid,
    output reg         done
);

    reg [3:0] index;

    // String: "Hello, it's me" (14 characters)
    function [7:0] get_char;
        input [3:0] i;
        case (i)
            4'd0:  get_char = 8'h48; // 'H'
            4'd1:  get_char = 8'h65; // 'e'
            4'd2:  get_char = 8'h6C; // 'l'
            4'd3:  get_char = 8'h6C; // 'l'
            4'd4:  get_char = 8'h6F; // 'o'
            4'd5:  get_char = 8'h2C; // ','
            4'd6:  get_char = 8'h20; // ' '
            4'd7:  get_char = 8'h69; // 'i'
            4'd8:  get_char = 8'h74; // 't'
            4'd9:  get_char = 8'h27; // '''
            4'd10: get_char = 8'h73; // 's'
            4'd11: get_char = 8'h20; // ' '
            4'd12: get_char = 8'h6D; // 'm'
            4'd13: get_char = 8'h65; // 'e'
            default: get_char = 8'h00;
        endcase
    endfunction

    localparam STR_LEN = 4'd14;

    localparam IDLE = 2'b00,
               SEND = 2'b01,
               DONE = 2'b10;

    reg [1:0] state;

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            state      <= IDLE;
            index      <= 4'd0;
            char_out   <= 8'd0;
            char_valid <= 1'b0;
            done       <= 1'b0;
        end else begin
            case (state)
                IDLE: begin
                    char_valid <= 1'b0;
                    done       <= 1'b0;
                    index      <= 4'd0;
                    if (en)
                        state <= SEND;
                end

                SEND: begin
                    char_out   <= get_char(index);
                    char_valid <= 1'b1;
                    if (index == STR_LEN - 1)
                        state <= DONE;
                    else
                        index <= index + 1'b1;
                end

                DONE: begin
                    char_valid <= 1'b0;
                    done       <= 1'b1;
                    state      <= IDLE;
                end

                default: state <= IDLE;
            endcase
        end
    end

endmodule
