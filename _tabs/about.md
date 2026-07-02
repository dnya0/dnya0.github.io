---
icon: fas fa-info-circle
order: 4
---

Hello, I'm <a href="https://github.com/dnya0">Nayeong Ahn.</a> It is a blog that summarizes what I've studied.

<br>

---

<div class="tl-wrap">

  <div class="tl-group">
    <div class="tl-heading">💼 Work Experience</div>
    <div class="tl-items">
      <div class="tl-item">
        <span class="tl-dot"></span>
        <span class="tl-date">Mar. '26 – <i>Jun. '30</i></span>
        <span class="tl-desc"><strong>TSE</strong> · Backend Engineer</span>
      </div>
      <div class="tl-item">
        <span class="tl-dot"></span>
        <span class="tl-date">May. '24 – Feb. '25</span>
        <span class="tl-desc"><strong>지란지교 데이터</strong> · 인턴</span>
      </div>
    </div>
  </div>

  <div class="tl-group">
    <div class="tl-heading">🎓 Education</div>
    <div class="tl-items">
      <div class="tl-item">
        <span class="tl-dot"></span>
        <span class="tl-date">Sep. '25 – Feb. '26</span>
        <span class="tl-desc">Wanted Poten-up 부트캠프</span>
      </div>
      <div class="tl-item">
        <span class="tl-dot"></span>
        <span class="tl-date">Mar. '20 – Feb. '24</span>
        <span class="tl-desc">공주대학교 컴퓨터공학부 소프트웨어전공</span>
      </div>
    </div>
  </div>

</div>

<style>
.tl-wrap { margin-top: 1rem; }
.tl-group { margin-bottom: 2rem; }
.tl-heading {
  font-size: 1.2rem;
  font-weight: 600;
  margin-bottom: 0.8rem;
}
.tl-items {
  position: relative;
  padding-left: 20px;
}
.tl-items::before {
  content: '';
  position: absolute;
  left: 3px;
  top: 22px;
  bottom: 22px;
  width: 2px;
  background: #cccccc95;
}
.tl-item {
  display: flex;
  align-items: center;
  padding: 10px 0;
}
 .tl-dot {
  position: absolute;
  left: 0;
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: var(--main-bg);
  border: 2px solid #ccc;
  z-index: 1;
}
.tl-date {
  font-size: 0.82rem;
  color: #aaa;
  font-family: monospace;
  min-width: 160px;
  padding-right: 16px;
}
.tl-desc {
  font-size: 0.95rem;
}
@media (max-width: 480px) {
  .tl-items::before {
    display: none;
  }
  .tl-dot {
    display: none;
  }
  .tl-item {
    flex-direction: column;
    align-items: flex-start;
    padding-left: 0;
  }
  .tl-date {
    min-width: unset;
    padding-right: 0;
    margin-bottom: 2px;
  }
}
</style>
