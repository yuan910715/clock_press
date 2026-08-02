切换至<a href='https://topyuan.top/clock/t'>中文</a>
## A manual update is required

This project has long relied on automatic push updates. However, the current version is no longer able to perform automatic upgrades due to firmware limitations. I have redesigned the program, and you will need to perform a manual upgrade. The upgraded version will still support automatic OTA updates  

Simply connect the chip to your computer using a USB cable, open https://topyuan.top/clock/installnew_en in the **Chrome browser**, and click the button to flash the new program 

`Please ensure that you connect the ESP32 chip to your computer using a USB data cable, as shown in the image below, and not directly connect it to the computer using a power cable (the power cable is only for power supply and cannot transmit data)`
![usb](/img/usb.jpg)

If you don't want to update manually, the current older version will continue to be available, but there will be no more OTA updates for new watch faces, features, etc  

***I strongly recommend that you update. The new version has added the following clockfaces and is continuously being updated via OTA.***
<div class="clockface-grid">
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

## Why

The ESP32 contains a partition table that divides the chip space into different regions. During OTA updates, only the app region can be updated, the partition table cannot be rewritten  

The older version used the default partition table, with the clock program running in app0 or app1. Both partitions were the same size, approximately 1310KB. Each time an update was pushed, the data was automatically written to the other partition and the system restarted 

`The current version v3.24 has reached 1304KB`
![old](/img/old_en.svg)


The new version redesigns the partition table. The clock program runs in app1, which is approximately 3014KB. Each time an update is pushed, it automatically restarts in app0. app0 only contains a rewritten update program, approximately 1MB, which is only used during updates  

![old](/img/new_en.svg)

`There is still plenty of space available for OTA updates under the new version's partition table`


## Thanks

Thank you for using this project, and I hope you enjoy it!

<div class="project-member-card">
  <img class="project-member-avatar" src="/img/member_feng.jpg" alt="Felix Feng" />
  <div class="project-member-info">
    <div class="project-member-name">Felix Feng</div>
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
