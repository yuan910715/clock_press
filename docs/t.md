Change to <a href='https://topyuan.top/clock/en/t'>English</a>
## 需进行一次手动更新

本项目长期以来，均为推送自动更新，现有版本因固件限制已无法完成自动升级，我已重新设计了程序，需要你进行一次手动升级，升级后的版本仍保持自动OTA更新 

你只需使用usb线，将芯片与你的电脑连接，使用**Chrome浏览器**打开https://topyuan.top/clock/installnew 点击按钮刷入新版程序即可 

`请确保你使用USB数据线连接ESP32芯片端至电脑，如下图，而非直接使用供电线连接至电脑(供电线仅供电使用，无法传输数据)`
![usb](/img/usb.jpg)

如果你不想手动更新，目前的旧版本也一直可用，只是后续再无新的表盘、功能等OTA更新  

**强烈建议你进行更新，新版本已增加如下表盘，并在持续OTA中..**
<div class="clockface-grid">
  <figure>
    <img src="/img/clockfaces/27_particle_clock.webp" alt="Particle-Time" loading="lazy" decoding="async" />
    <figcaption>Particle-Time</figcaption>
  </figure>
  <figure>
    <img src="/img/clockfaces/26_zelda_sunrise.webp" alt="Zelda-Sunrise" loading="lazy" decoding="async" />
    <figcaption>Zelda-Sunrise</figcaption>
  </figure>
  <figure>
    <img src="/img/clockfaces/25_GTA_VI.webp" alt="GTA VI" loading="lazy" decoding="async" />
    <figcaption>GTA VI</figcaption>
  </figure>
  <figure>
    <img src="/img/clockfaces/24_rainy_window.webp" alt="Rainy Window" loading="lazy" decoding="async" />
    <figcaption>Rainy Window</figcaption>
  </figure>
</div>

## 为什么

ESP32包含一个分区表，将芯片空间划分为不同区域，在OTA更新时，仅能更新app区域，无法重写分区表 

旧版本使用默认分区表，时钟程序运行在app0或app1中，两个分区大小一致，约为1310KB，每次推送更新时，自动写入另一分区并重启 

`现有版本v3.24已经达到1304KB`
![old](/img/old.svg)


新版本重新设计了分区表，时钟程序运行在app1中，约为3014KB，每次推送更新时，自动重启至app0，app0只有重写的更新程序，约1MB，仅在更新时使用 

![old](/img/new.svg)

`在新版本分区表下，还有大量空间可以用于OTA`

## 感谢

感谢你的使用，希望你喜欢本项目！

<div class="project-member-card">
  <img class="project-member-avatar" src="/img/member_feng.jpg" alt="Felix Feng" />
  <div class="project-member-info">
    <div class="project-member-name">冯雪原</div>
    <div class="project-member-title"><a href="mailto:admin@topyuan.top">admin@topyuan.top</a></div>
    <a class="project-member-link" href="https://github.com/yuan910715" target="_blank" rel="noreferrer">GitHub</a>
  </div>
</div>

<style scoped>
.project-member-card {
  display: flex;
  align-items: center;
  gap: 28px;
  margin-top: 20px;
}

.project-member-avatar {
  width: 220px;
  aspect-ratio: 1 / 1.12;
  border-radius: 12px;
  object-fit: cover;
  object-position: 58% top;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.12);
}

.project-member-info {
  min-width: 0;
}

.project-member-name {
  font-size: 24px;
  font-weight: 700;
  line-height: 1.3;
}

.project-member-title {
  margin-top: 6px;
  color: var(--vp-c-text-2);
}

.project-member-link {
  display: inline-block;
  margin-top: 14px;
  font-weight: 600;
}

@media (max-width: 640px) {
  .project-member-card {
    align-items: flex-start;
    flex-direction: column;
    gap: 16px;
  }

  .project-member-avatar {
    width: min(100%, 240px);
  }
}
.clockface-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
  gap: 16px;
}

.clockface-grid img {
  width: 100%;
  max-width: 220px;
  height: auto;
  justify-self: center;
}

.clockface-grid figure {
  margin: 0;
  min-width: 0;
  text-align: center;
}

.clockface-grid figcaption {
  margin-top: 8px;
  color: var(--vp-c-text-2);
  font-size: 13px;
  line-height: 1.35;
}
</style>
