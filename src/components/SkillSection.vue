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
    this.isHovered = false;
    this.isDragged = false;
    this.hasBounced = false;
    this.isSeparated = false;
    this.popup = document.createElement("div");
    this.popup.className = "popup";
    document.body.appendChild(this.popup);
  }

  draw(ctx: CanvasRenderingContext2D) {
    ctx.beginPath();
    ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
    ctx.fillStyle = "#ffcc00";
    ctx.fill();
    ctx.closePath();
    ctx.fillStyle = "#000";
    ctx.font = "bold 14px Arial";
    ctx.textAlign = "center";
    ctx.textBaseline = "middle";
    ctx.fillText(this.skill.name, this.x, this.y);
  }

  update(ctx: CanvasRenderingContext2D, canvas: HTMLCanvasElement) {
    if (this.isDragged) {
      // Không áp dụng vận tốc khi kéo
    } else if (!this.isHovered) {
      if (!this.hasBounced) {
        this.y += this.dy;
        this.dy += gravity;
        if (this.y + this.radius > canvas.height) {
          this.y = canvas.height - this.radius;
          this.dy = -Math.sqrt(2 * gravity * (canvas.height / 2));
          this.dx = Math.random() * 6 - 3;
          this.hasBounced = true;
        }
      } else {
        this.x += this.dx;
        this.y += this.dy;
        this.dx *= friction;
        this.dy *= friction;

        if (
          this.y <= canvas.height / 2 &&
          this.hasBounced &&
          !this.isSeparated
        ) {
          this.isSeparated = true;
        }

        if (Math.abs(this.dx) < minSpeed)
          this.dx = minSpeed * Math.sign(this.dx);
        if (Math.abs(this.dy) < minSpeed)
          this.dy = minSpeed * Math.sign(this.dy);

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

  showPopup() {
    this.popup.style.left = `${this.x + this.radius}px`;
    this.popup.style.top = `${this.y - this.radius}px`;
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
const gravity = 0.1;
const friction = 0.99;
const minSpeed = 2;

// Danh sách skills
const skills: Skill[] = [
  { name: "HTML", desc: "Ngôn ngữ đánh dấu để tạo cấu trúc web" },
  { name: "CSS", desc: "Ngôn ngữ định kiểu cho giao diện web" },
  { name: "JavaScript", desc: "Ngôn ngữ lập trình cho web động" },
  { name: "React", desc: "Thư viện JS để xây dựng UI" },
  { name: "Node.js", desc: "Môi trường chạy JS phía server" },
  { name: "Python", desc: "Ngôn ngữ lập trình đa năng" },
  { name: "Figma", desc: "Công cụ thiết kế UI/UX" },
  { name: "UI/UX", desc: "Thiết kế trải nghiệm người dùng" },
  { name: "Vue", desc: "Framework JS để xây dựng UI" },
  { name: "Nuxt", desc: "Framework dựa trên Vue cho SSR" },
  { name: "Nest", desc: "Framework Node.js cho backend" },
  { name: "Next", desc: "Framework React cho SSR và SSG" },
  { name: "Git", desc: "Hệ thống quản lý phiên bản" },
  { name: "Teamwork", desc: "Kỹ năng làm việc nhóm" },
  { name: "Laravel", desc: "Framework PHP cho phát triển web" },
];

// Mảng chứa các bóng
const balls: Ball[] = [];
let draggedBall: Ball | null = null;

onMounted(() => {
  const canvas = ballCanvas.value!;
  const ctx = canvas.getContext("2d")!;
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;

  // Hàm lấy tọa độ chuột tương đối với canvas
  const getMousePos = (event: MouseEvent) => {
    const rect = canvas.getBoundingClientRect();
    return {
      x: event.clientX - rect.left,
      y: event.clientY - rect.top,
    };
  };

  // Khởi tạo các bóng
  function init() {
    for (let i = 0; i < skills.length; i++) {
      const radius = 50;
      const x = Math.random() * (canvas.width - radius * 2) + radius;
      const y = -radius;
      const dy = 1;
      balls.push(new Ball(skills[i], x, y, dy, radius));
    }
  }

  // Hàm animate
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

  // Sự kiện chuột
  canvas.addEventListener("mousedown", (e) => {
    const mousePos = getMousePos(e);
    const mouseX = mousePos.x;
    const mouseY = mousePos.y;

    balls.forEach((ball) => {
      const dist = Math.sqrt((mouseX - ball.x) ** 2 + (mouseY - ball.y) ** 2);
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
    const mouseX = mousePos.x;
    const mouseY = mousePos.y;

    if (draggedBall) {
      draggedBall.x = mouseX;
      draggedBall.y = mouseY;
    } else {
      balls.forEach((ball) => {
        const dist = Math.sqrt((mouseX - ball.x) ** 2 + (mouseY - ball.y) ** 2);
        if (dist < ball.radius && !ball.isDragged) {
          if (!ball.isHovered) {
            ball.isHovered = true;
            ball.showPopup();
          }
        } else {
          if (ball.isHovered) {
            ball.isHovered = false;
            ball.hidePopup();
          }
        }
      });
    }
  });

  canvas.addEventListener("mouseup", () => {
    if (draggedBall) {
      draggedBall.dx = Math.random() * 6 - 3;
      draggedBall.dy = Math.random() * 6 - 3;
      draggedBall.isDragged = false;
      draggedBall.hasBounced = true;
      draggedBall.isSeparated = true;
      draggedBall = null;
    }
  });

  // Xử lý resize
  const handleResize = () => {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
  };
  window.addEventListener("resize", handleResize);

  // Khởi chạy
  init();
  animate();

  // Cleanup khi component bị hủy
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
