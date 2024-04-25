# GIT-TEST
#!/bin/bash

# Function to draw the game board
draw_board() {
    clear
    for ((i = 0; i < height; i++)); do
        for ((j = 0; j < width; j++)); do
            if [[ ${board[i*width+j]} == 1 ]]; then
                echo -n "▓▓"
            else
                echo -n "░░"
            fi
        done
        echo ""
    done
}

# Function to generate a new piece
generate_piece() {
    current_piece=${pieces[$(($RANDOM % ${#pieces[@]}))]}
    piece_rotation=$(( $RANDOM % ${#current_piece[@]} ))
    piece_x=$(( (width - ${#current_piece[0]}) / 2 ))
    piece_y=0
}

# Function to check if a piece can move
can_move() {
    local dx=$1
    local dy=$2
    for ((i = 0; i < ${#current_piece[@]}; i++)); do
        for ((j = 0; j < ${#current_piece[0]}; j++)); do
            local x=$((piece_x + j + dx))
            local y=$((piece_y + i + dy))
            if [[ ${current_piece[i]:j:1} == 1 && (x < 0 || x >= width || y >= height || (y >= 0 && ${board[y*width+x]} == 1)) ]]; then
                return 1
            fi
        done
    done
    return 0
}

# Function to merge current piece into the board
merge_piece() {
    for ((i = 0; i < ${#current_piece[@]}; i++)); do
        for ((j = 0; j < ${#current_piece[0]}; j++)); do
            local x=$((piece_x + j))
            local y=$((piece_y + i))
            if [[ ${current_piece[i]:j:1} == 1 && y -ge 0 ]]; then
                board[y*width+x]=1
            fi
        done
    done
}

# Function to check for lines to clear
check_lines() {
    for ((i = height - 1; i >= 0; i--)); do
        local line=1
        for ((j = 0; j < width; j++)); do
            if [[ ${board[i*width+j]} != 1 ]]; then
                line=0
                break
            fi
        done
        if [[ $line == 1 ]]; then
            for ((k = i; k > 0; k--)); do
                for ((j = 0; j < width; j++)); do
                    board[k*width+j]=${board[(k-1)*width+j]}
                done
            done
            for ((j = 0; j < width; j++)); do
                board[j]=0
            done
            ((i++))
        fi
    done
}

# Initialize game variables
width=10
height=20
declare -a board
for ((i = 0; i < width * height; i++)); do
    board[i]=0
done

# Define Tetris pieces
pieces=(
    "1111"
    "1110 0010"
    "1100 0110"
    "0110 1100"
    "0110 0011"
    "1100 0110"
    "0010 0010 0010 0010"
)

# Main game loop
while true; do
    generate_piece
    draw_board
    while can_move 0 1; do
        read -s -n 1 -t 0.1 key
        case "$key" in
            'a') if can_move -1 0; then ((piece_x--)); fi ;;
            'd') if can_move 1 0; then ((piece_x++)); fi ;;
            's') if can_move 0 1; then ((piece_y++)); fi ;;
            'w') rotate_piece ;;
        esac
        draw_board
    done
    merge_piece
    check_lines
    draw_board
done
