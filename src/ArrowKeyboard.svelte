<script>
  import { onMount } from 'svelte';
  // @ts-ignore
  import ROSLIB from 'roslib';

  let direction = 'None';
  let turtleX = 0;
  let turtleY = 0;
  let turtleTheta = 0;

  // ROS setup
  const ros = new ROSLIB.Ros({
    // Insert IP
    url: 'ws://XXX.XXX.XX.XX:9090'
  });

  ros.on('connection', () => {
    console.log('Connected to websocket server.');
  });

  // @ts-ignore
  ros.on('error', (error) => {
    console.log('Error connecting to websocket server: ', error);
  });

  ros.on('close', () => {
    console.log('Connection to websocket server closed.');
  });

  // Create a topic object
  const cmdVel = new ROSLIB.Topic({
    ros: ros,
    name: '/turtle1/cmd_vel',
    messageType: 'geometry_msgs/Twist'
  });

  const poseListener = new ROSLIB.Topic({
    ros: ros,
    name: '/turtle1/pose',
    messageType: 'turtlesim/Pose'
  });

  function drawTurtle() {
    const canvas = document.getElementById('turtleCanvas');
    // @ts-ignore
    const ctx = canvas.getContext('2d');

    if (!ctx) {
      return;
    }

    // Clear the canvas
    // @ts-ignore
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Draw the turtle as a circle
    ctx.beginPath();
    // @ts-ignore
    ctx.arc(turtleX * 50, canvas.height - turtleY * 50, 10, 0, 2 * Math.PI);
    ctx.fillStyle = 'blue';
    ctx.fill();

    // Draw the direction arrow
    ctx.beginPath();
    // @ts-ignore
    ctx.moveTo(turtleX * 50, canvas.height - turtleY * 50);
    ctx.lineTo(
      turtleX * 50 + 20 * Math.cos(turtleTheta),
      // @ts-ignore
      canvas.height - turtleY * 50 - 20 * Math.sin(turtleTheta)
    );
    ctx.strokeStyle = 'red';
    ctx.lineWidth = 2;
    ctx.stroke();
  }

  onMount(() => {
    // @ts-ignore
    poseListener.subscribe((message) => {
      turtleX = message.x;
      turtleY = message.y;
      turtleTheta = message.theta;
      drawTurtle();
    });

    drawTurtle();
  });

  // Function to publish movement commands
  // @ts-ignore
  function publishCommand(linear, angular) {
    const twist = new ROSLIB.Message({
      linear: {
        x: linear,
        y: 0.0,
        z: 0.0
      },
      angular: {
        x: 0.0,
        y: 0.0,
        z: angular
      }
    });
    cmdVel.publish(twist);
  }

  // @ts-ignore
  function handleButtonClick(dir) {
    switch (dir) {
      case 'Up':
        publishCommand(1.0, 0.0);
        break;
      case 'Down':
        publishCommand(-1.0, 0.0);
        break;
      case 'Left':
        publishCommand(0.0, 1.0);
        break;
      case 'Right':
        publishCommand(0.0, -1.0);
        break;
    }
    direction = dir;
  }
</script>

<style>
  .container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100vh;
  }

  .canvas-container {
    margin-bottom: 20px;
  }

  .buttons {
    display: grid;
    grid-template-columns: repeat(3, 50px);
    grid-template-rows: repeat(3, 50px);
    gap: 10px;
  }

  .button {
    display: flex;
    justify-content: center;
    align-items: center;
    border: 1px solid #ccc;
    border-radius: 5px;
    cursor: pointer;
    user-select: none;
  }

  .button:active {
    background-color: #ddd;
  }

  .direction {
    grid-column: 2 / 3;
    grid-row: 2 / 3;
  }

  .up {
    grid-column: 2 / 3;
    grid-row: 1 / 2;
  }

  .down {
    grid-column: 2 / 3;
    grid-row: 3 / 4;
  }

  .left {
    grid-column: 1 / 2;
    grid-row: 2 / 3;
  }

  .right {
    grid-column: 3 / 4;
    grid-row: 2 / 3;
  }

  #turtleCanvas {
    width: 400px;
    height: 400px;
    border: 1px solid #ccc;
    background-color: #f0f0f0;
  }
</style>

<div class="container">
  <div class="canvas-container">
    <canvas id="turtleCanvas" width="400" height="400"></canvas>
  </div>
  <div class="buttons">
    <div class="button up" on:click={() => handleButtonClick('Up')}>↑</div>
    <div class="button left" on:click={() => handleButtonClick('Left')}>←</div>
    <div class="button direction">{direction}</div>
    <div class="button right" on:click={() => handleButtonClick('Right')}>→</div>
    <div class="button down" on:click={() => handleButtonClick('Down')}>↓</div>
  </div>
</div>
