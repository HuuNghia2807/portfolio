<template>
  <section class="skills" id="skills">
    <h2 class="title">My <span>Skills</span></h2>
    <canvas ref="ballCanvas" id="ballCanvas"></canvas>
  </section>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from "vue";

// Ref cho canvas
const ballCanvas = ref<HTMLCanvasElement | null>(null);

// Định nghĩa interface cho skill
interface Skill {
  name: string;
  desc: string;
}

// Định nghĩa class Ball
class Ball {
  skill: Skill;
  x: number;
  y: number;
  dx: number;
  dy: number;
  radius: number;
  fontSize: number;
  isHovered: boolean;
  isDragged: boolean;
  hasBounced: boolean;
  isSeparated: boolean;
  popup: HTMLDivElement;

  constructor(skill: Skill, x: number, y: number, dy: number, radius: number) {
    this.skill = skill;
    this.x = x;
    this.y = y;
    this.dx = 0;
    this.dy = dy;
    this.radius = radius;
    this.fontSize = this.calculateFontSize();
    this.isHovered = false;
    this.isDragged = false;
    this.hasBounced = false;
    this.isSeparated = false;
    this.popup = document.createElement("div");
    this.popup.className = "popup";
    document.body.appendChild(this.popup);
  }

  calculateFontSize(): number {
    const fontSize = this.radius * 0.25;
    return Math.max(8, Math.min(fontSize, 20));
  }

  updateSize(newRadius: number) {
    this.radius = newRadius;
    this.fontSize = this.calculateFontSize();
  }

  draw(ctx: CanvasRenderingContext2D) {
    const gradient = ctx.createLinearGradient(
      this.x - this.radius,
      this.y - this.radius,
      this.x + this.radius,
      this.y + this.radius
    );
    gradient.addColorStop(0, "#00f");
    gradient.addColorStop(1, "#0ff");

    ctx.beginPath();
    ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
    ctx.fillStyle = "transparent";
    ctx.fill();
    ctx.lineWidth = 8;
    ctx.strokeStyle = gradient;
    ctx.stroke();
    ctx.closePath();

    ctx.fillStyle = "#fff";
    ctx.font = `bold ${this.fontSize}px Arial`;
    ctx.textAlign = "center";
    ctx.textBaseline = "middle";
    ctx.fillText(this.skill.name, this.x, this.y);
  }

  update(ctx: CanvasRenderingContext2D, canvas: HTMLCanvasElement) {
    if (!this.isDragged) {
      if (!this.hasBounced) {
        this.y += this.dy;
        this.dy += gravity;
        if (this.y + this.radius > canvas.height) {
          this.y = canvas.height - this.radius;
          this.dy = -0.8; // Tốc độ bật lên là -0.8
          this.dx = Math.random() < 0.5 ? 0.8 : -0.8; // Tốc độ ngang là 0.8 hoặc -0.8
          this.hasBounced = true;
        }
      } else {
        this.x += this.dx;
        this.y += this.dy;
        // Bỏ friction và minSpeed để tốc độ không giảm dần
        if (
          this.y <= canvas.height / 2 &&
          this.hasBounced &&
          !this.isSeparated
        ) {
          this.isSeparated = true;
        }

        if (this.x - this.radius < 0 || this.x + this.radius > canvas.width) {
          this.dx *= -1;
        }
        if (this.y - this.radius < 0 || this.y + this.radius > canvas.height) {
          this.dy *= -1;
        }
      }
    }
    this.draw(ctx);
  }

  checkCollision(otherBall: Ball) {
    if (!this.isSeparated || !otherBall.isSeparated) return;

    const dx = otherBall.x - this.x;
    const dy = otherBall.y - this.y;
    const distance = Math.sqrt(dx * dx + dy * dy);
    const minDistance = this.radius + otherBall.radius;

    if (
      distance < minDistance &&
      !this.isHovered &&
      !otherBall.isHovered &&
      !this.isDragged &&
      !otherBall.isDragged
    ) {
      const angle = Math.atan2(dy, dx);
      const overlap = minDistance - distance;
      const pushX = (Math.cos(angle) * overlap) / 2;
      const pushY = (Math.sin(angle) * overlap) / 2;

      this.x -= pushX;
      this.y -= pushY;
      otherBall.x += pushX;
      otherBall.y += pushY;

      this.dx *= -1;
      this.dy *= -1;
      otherBall.dx *= -1;
      otherBall.dy *= -1;
    }
  }

  showPopup(canvasRect: DOMRect) {
    this.popup.style.left = `${canvasRect.left + this.x + this.radius}px`;
    this.popup.style.top = `${canvasRect.top + this.y - this.radius}px`;
    this.popup.textContent = this.skill.desc;
    this.popup.style.display = "block";
    this.popup.style.zIndex = "20";
  }

  hidePopup() {
    this.popup.style.display = "none";
    this.popup.style.zIndex = "10";
  }
}

// Các hằng số
const gravity = 0.05;
const friction = 1; // Đặt friction = 1 để không giảm tốc độ
// Bỏ minSpeed vì không cần thiết khi không có giảm dần

// Danh sách skills
const skills: Skill[] = [
  { name: "HTML, CSS", desc: "" },
  { name: "Javascript", desc: "" },
  { name: "Typescript", desc: "" },
  { name: "VueJS, NuxtJS", desc: "" },
  { name: "PHP", desc: "" },
  { name: "Git", desc: "" },
  { name: "Communicate", desc: "" },
  { name: "Teamwork", desc: "" },
  { name: "Responsibility", desc: "" },
];

// Mảng chứa các bóng
const balls: Ball[] = [];
let draggedBall: Ball | null = null;

onMounted(() => {
  const canvas = ballCanvas.value!;
  const ctx = canvas.getContext("2d")!;

  const calculateRadius = (canvasWidth: number): number => {
    let baseRadius: number;
    if (canvasWidth <= 430) {
      baseRadius = canvasWidth * 0.04;
    } else {
      baseRadius = canvasWidth * 0.05;
    }
    return Math.max(30, Math.min(baseRadius, 75));
  };

  const updateCanvasSize = () => {
    const rect = canvas.getBoundingClientRect();
    canvas.width = rect.width;
    canvas.height = rect.height;

    const newRadius = calculateRadius(canvas.width);
    balls.forEach((ball) => ball.updateSize(newRadius));
  };

  updateCanvasSize();

  const getMousePos = (event: MouseEvent) => {
    const rect = canvas.getBoundingClientRect();
    return {
      x: event.clientX - rect.left,
      y: event.clientY - rect.top,
    };
  };

  function init() {
    balls.length = 0;
    const radius = calculateRadius(canvas.width);
    for (let i = 0; i < skills.length; i++) {
      const x = Math.random() * (canvas.width - radius * 2) + radius;
      const y = -radius;
      const dy = 0.8; // Tốc độ rơi ban đầu là 0.8
      balls.push(new Ball(skills[i], x, y, dy, radius));
    }
  }

  function animate() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    for (let i = 0; i < balls.length; i++) {
      for (let j = i + 1; j < balls.length; j++) {
        balls[i].checkCollision(balls[j]);
      }
    }

    balls.forEach((ball) => ball.update(ctx, canvas));
    requestAnimationFrame(animate);
  }

  canvas.addEventListener("mousedown", (e) => {
    const mousePos = getMousePos(e);
    balls.forEach((ball) => {
      const dist = Math.sqrt(
        (mousePos.x - ball.x) ** 2 + (mousePos.y - ball.y) ** 2
      );
      if (dist < ball.radius) {
        draggedBall = ball;
        ball.isDragged = true;
        ball.isHovered = false;
        ball.hidePopup();
      }
    });
  });

  canvas.addEventListener("mousemove", (e) => {
    const mousePos = getMousePos(e);
    const canvasRect = canvas.getBoundingClientRect();

    if (draggedBall) {
      draggedBall.x = mousePos.x;
      draggedBall.y = mousePos.y;
    } else {
      balls.forEach((ball) => {
        const dist = Math.sqrt(
          (mousePos.x - ball.x) ** 2 + (mousePos.y - ball.y) ** 2
        );
        if (dist < ball.radius) {
          if (!ball.isHovered) {
            ball.isHovered = true;
            // ball.showPopup(canvasRect);
          }
        } else if (ball.isHovered) {
          ball.isHovered = false;
          ball.hidePopup();
        }
      });
    }
  });

  canvas.addEventListener("mouseup", () => {
    if (draggedBall) {
      draggedBall.dx = Math.random() < 0.5 ? 0.8 : -0.8; // Tốc độ ban đầu là 0.8 hoặc -0.8
      draggedBall.dy = Math.random() < 0.5 ? 0.8 : -0.8; // Tốc độ ban đầu là 0.8 hoặc -0.8
      draggedBall.isDragged = false;
      draggedBall.hasBounced = true;
      draggedBall.isSeparated = true;
      draggedBall = null;
    }
  });

  const handleResize = () => {
    updateCanvasSize();
    init();
    const canvasRect = canvas.getBoundingClientRect();
    // balls.forEach((ball) => {
    //   if (ball.isHovered) ball.showPopup(canvasRect);
    // });
  };
  window.addEventListener("resize", handleResize);

  init();
  animate();

  onUnmounted(() => {
    window.removeEventListener("resize", handleResize);
    balls.forEach((ball) => document.body.removeChild(ball.popup));
  });
});
</script>

<style lang="scss" scoped>
.skills {
  min-height: auto;
  padding-bottom: 7rem;
  background-color: var(--bg-color);
  position: relative;

  #ballCanvas {
    width: 100%;
    height: 100%;
  }

  .popup {
    position: absolute;
    background: #fff;
    padding: 10px;
    border-radius: 5px;
    display: none;
    z-index: 10;
    max-width: 200px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
  }
}
</style>
