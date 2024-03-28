<template>
  <div id="game">
  </div>
</template>

<script setup lang="ts">
import {Game, AUTO, Scale, Scene} from 'phaser';
import nipplejs from 'nipplejs';
import {onMounted} from "vue";

let hair_type = 'spikey'
let player: Phaser.Physics.Arcade.Sprite;
let player_hair: Phaser.Physics.Arcade.Sprite;
let player_tool: Phaser.Physics.Arcade.Sprite;
let sheep;

class MainScene extends Scene {
  constructor() {
    super({key: 'MainScene'})
  }

  preload() {
    this.load.spritesheet('sheep', 'assets/sheep.png', {frameWidth: 32, frameHeight: 32});

    this.load.spritesheet('player_idle', 'assets/Human/IDLE/base_idle_strip9.png', {frameWidth: 96, frameHeight: 64});
    this.load.spritesheet('player_idle_hair', `assets/Human/IDLE/${hair_type}hair_idle_strip9.png`, {
      frameWidth: 96,
      frameHeight: 64
    });
    this.load.spritesheet('player_idle_tool', `assets/Human/IDLE/tools_idle_strip9.png`, {
      frameWidth: 96,
      frameHeight: 64
    });
    this.load.spritesheet('player_sword_tool', `assets/Human/IDLE/tools_idle_strip9.png`, {
      frameWidth: 96,
      frameHeight: 64
    });

    this.load.spritesheet('player_walk', 'assets/Human/WALKING/base_walk_strip8.png', {
      frameWidth: 96,
      frameHeight: 64
    });
    this.load.spritesheet('player_walk_hair', `assets/Human/IDLE/${hair_type}hair_walk_strip8.png`, {
      frameWidth: 96,
      frameHeight: 64
    });

    this.load.spritesheet('player_walk_tool', `assets/Human/WALKING/tools_walk_strip8.png`, {
      frameWidth: 96,
      frameHeight: 64
    });

  }

  create() {
    const screenWidth = this.scale.width;
    const screenHeight = this.scale.height;
    const screenCenterX = screenWidth / 2;
    const controlsAreaHeight = screenHeight * 0.2;
    const gameAreaHeight = screenHeight - controlsAreaHeight;
    const minX = 0; // Leftmost point
    const maxX = screenWidth; // Rightmost point
    const minY = gameAreaHeight - 100; // Bottom of the game area minus 100 pixels
    const maxY = gameAreaHeight; // Top of the game area

    // adds the player, platform, and controls
    player = this.physics.add.sprite(screenCenterX, gameAreaHeight - 24, 'player_idle');
    player_hair = this.physics.add.sprite(screenCenterX, gameAreaHeight - 24, 'player_idle_hair');
    player_tool = this.physics.add.sprite(screenCenterX, gameAreaHeight - 24, 'player_idle_tool');
    sheep = this.physics.add.sprite(screenCenterX, gameAreaHeight - 100, 'sheep');

    sheep.anims.create({
      key: "idle",
      frames: this.anims.generateFrameNumbers('sheep', {start: 0, end: 3}),
      frameRate: 10,
      repeat: -1
    })
    sheep.anims.play('idle')

    player_hair.anims.create({
      key: "idle",
      frames: this.anims.generateFrameNumbers('player_idle_hair', {start: 0, end: 8}),
      frameRate: 10,
      repeat: -1,
    })
    player_hair.anims.create({
      key: "walk",
      frames: this.anims.generateFrameNumbers('player_idle_hair', {start: 0, end: 7}),
      frameRate: 25,
      repeat: -1,
    })

    player_tool.anims.create({
      key: "idle",
      frames: this.anims.generateFrameNumbers('player_idle_tool', {start: 0, end: 8}),
      frameRate: 10,
      repeat: -1,
    })
    player_tool.anims.create({
      key: "walk",
      frames: this.anims.generateFrameNumbers('player_walk_tool', {start: 0, end: 7}),
      frameRate: 25,
      repeat: -1,
    })

    if (!this.anims.exists('idle')) {
      this.anims.create({
        key: "idle",
        frames: this.anims.generateFrameNumbers('player_idle', {start: 0, end: 8}),
        frameRate: 10,
        repeat: -1,
      });
    }
    if (!this.anims.exists('walk')) {
      this.anims.create({
        key: "walk",
        frames: this.anims.generateFrameNumbers('player_walk', {start: 0, end: 7}),
        frameRate: 15,
        repeat: -1,
      });
    }
    player.anims.play('idle')
    player_hair.anims.play('idle')
    player_tool.anims.play('idle')
    this.cameras.main.startFollow(player, true).setZoom(2,2)

  }

  update() {
  }
}

function launch() {
  return new Game({
    type: AUTO,
    scale: {
      mode: Scale.RESIZE,
      width: window.innerWidth * window.devicePixelRatio,
      autoCenter: Scale.CENTER_BOTH,
      height: window.innerHeight * window.devicePixelRatio,
    },
    pixelArt: true,
    parent: "game",
    backgroundColor: "#201726",
    physics: {
      default: "arcade",
    },
    scene: MainScene,
  });
}

onMounted(() => {
  launch();
  const el = document.getElementById('game')
  const joystickManager = nipplejs.create({zone: el});

  joystickManager.on('move', (data: nipplejs.EventData, output: nipplejs.JoystickOutputData) => {
    const force = Math.min(output.force, 1);
    const angle = output.angle.radian;
    const speed = 150 * force;
    const vector = output.vector
    player.setVelocity(speed * Math.cos(angle), speed * Math.sin(angle) * -1);
    player_hair.setVelocity(speed * Math.cos(angle), speed * Math.sin(angle) * -1);
    player_tool.setVelocity(speed * Math.cos(angle), speed * Math.sin(angle) * -1);

    if (player.anims.currentAnim.key === 'idle') {
      player.anims.play("walk")
      player_tool.anims.play("walk")
      player_hair.anims.play("walk")
    }
    if (vector.x < 0 && !player.flipX) {
      player.setFlipX(1)
      player_hair.setFlipX(1)
      player_tool.setFlipX(1)
    }
    if (vector.x > 0 && player.flipX) {
      player.setFlipX(0)
      player_hair.setFlipX(0)
      player_tool.setFlipX(0)
    }

  });
  joystickManager.on('end', () => {
    player.setVelocity(0, 0);
    player_hair.setVelocity(0, 0);
    player_tool.setVelocity(0, 0);
    player.anims.play("idle")
    player_hair.anims.play("idle")
    player_tool.anims.play("idle")
  })
})

</script>

<style scoped>
#game {
  height: 100%;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 0;
}
</style>