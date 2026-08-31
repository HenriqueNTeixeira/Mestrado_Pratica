# dissertacao_ws — ambiente

## Iniciar o ambiente
export USER_NAME="$USER"
xhost +local:docker
cd ~/Desktop/MESTRADO/dissertacao_ws
docker compose run --rm dissertacao bash

## Build
Usar `catkin build`, não `catkin_make` — a imagem lar-gazebo:noetic já foi
construída com catkin_tools, misturar as duas dá conflito no build space.

cd /ws
catkin build
source devel/setup.bash
