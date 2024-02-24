<script lang="ts">
export default {
  name: "FireFlies"
};
</script>

<template>
  <div class="w-full h-full">
    <div class="firefly"></div>
    <div class="firefly"></div>
    <div class="firefly"></div>
    <div class="firefly"></div>
    <div class="firefly"></div>
    <div class="firefly"></div>
    <div class="firefly"></div>
    <div class="firefly"></div>
    <div class="firefly"></div>
    <div class="firefly"></div>
    <div class="firefly"></div>
    <div class="firefly"></div>
    <div class="firefly"></div>
    <div class="firefly"></div>
    <div class="firefly"></div>
    <div class="firefly"></div>
    <div class="firefly"></div>
    <div class="firefly"></div>
    <div class="firefly"></div>
    <div class="firefly"></div>
    <div class="firefly"></div>
  </div>
</template>

<style>
.firefly {
  position: fixed;
  left: 50%;
  top: 50%;
  width: 0.4vw;
  height: 0.4vw;
  margin: -0.2vw 0 0 9.8vw;
  animation: ease 200s alternate infinite;
  pointer-events: none;
  z-index: -1;
}
.firefly::before,
.firefly::after {
  content: "";
  position: absolute;
  width: 100%;
  height: 100%;
  border-radius: 50%;
  transform-origin: -10vw;
}
.firefly::before {
  background: black;
  opacity: 0.4;
  animation: drift ease alternate infinite;
}
.firefly::after {
  background: white;
  opacity: 0;
  box-shadow: 0 0 0vw 0vw yellow;
  animation: drift ease alternate infinite, flash ease infinite;
}

.firefly:nth-child(1) {
  animation-name: move1;
}
.firefly:nth-child(1)::before {
  animation-duration: 13s;
}
.firefly:nth-child(1)::after {
  animation-duration: 13s, 6987ms;
  animation-delay: 0ms, 7138ms;
}

@keyframes move1 {
  0% {
    transform: translateX(44vw) translateY(7vh) scale(0.68);
  }
  5.8823529412% {
    transform: translateX(-35vw) translateY(27vh) scale(0.58);
  }
  11.7647058824% {
    transform: translateX(-13vw) translateY(-41vh) scale(0.81);
  }
  17.6470588235% {
    transform: translateX(-10vw) translateY(41vh) scale(0.52);
  }
  23.5294117647% {
    transform: translateX(-19vw) translateY(-20vh) scale(0.38);
  }
  29.4117647059% {
    transform: translateX(-10vw) translateY(41vh) scale(0.41);
  }
  35.2941176471% {
    transform: translateX(29vw) translateY(-27vh) scale(0.95);
  }
  41.1764705882% {
    transform: translateX(35vw) translateY(-30vh) scale(0.78);
  }
  47.0588235294% {
    transform: translateX(18vw) translateY(-17vh) scale(0.48);
  }
  52.9411764706% {
    transform: translateX(-38vw) translateY(-21vh) scale(0.99);
  }
  58.8235294118% {
    transform: translateX(2vw) translateY(-34vh) scale(0.8);
  }
  64.7058823529% {
    transform: translateX(5vw) translateY(-9vh) scale(0.38);
  }
  70.5882352941% {
    transform: translateX(12vw) translateY(50vh) scale(0.8);
  }
  76.4705882353% {
    transform: translateX(-4vw) translateY(-42vh) scale(0.98);
  }
  82.3529411765% {
    transform: translateX(-6vw) translateY(38vh) scale(0.75);
  }
  88.2352941176% {
    transform: translateX(-32vw) translateY(-6vh) scale(0.85);
  }
  94.1176470588% {
    transform: translateX(11vw) translateY(16vh) scale(0.27);
  }
  100% {
    transform: translateX(-35vw) translateY(-36vh) scale(0.27);
  }
}
.firefly:nth-child(2) {
  animation-name: move2;
}
.firefly:nth-child(2)::before {
  animation-duration: 11s;
}
.firefly:nth-child(2)::after {
  animation-duration: 11s, 8051ms;
  animation-delay: 0ms, 2531ms;
}

@keyframes move2 {
  0% {
    transform: translateX(-15vw) translateY(33vh) scale(0.94);
  }
  4% {
    transform: translateX(-40vw) translateY(40vh) scale(0.89);
  }
  8% {
    transform: translateX(-47vw) translateY(10vh) scale(0.98);
  }
  12% {
    transform: translateX(-20vw) translateY(37vh) scale(0.99);
  }
  16% {
    transform: translateX(-5vw) translateY(-40vh) scale(0.98);
  }
  20% {
    transform: translateX(-26vw) translateY(-25vh) scale(0.65);
  }
  24% {
    transform: translateX(14vw) translateY(24vh) scale(0.94);
  }
  28% {
    transform: translateX(1vw) translateY(-46vh) scale(0.84);
  }
  32% {
    transform: translateX(-1vw) translateY(-26vh) scale(0.29);
  }
  36% {
    transform: translateX(5vw) translateY(12vh) scale(0.67);
  }
  40% {
    transform: translateX(-35vw) translateY(-48vh) scale(0.53);
  }
  44% {
    transform: translateX(-41vw) translateY(25vh) scale(0.85);
  }
  48% {
    transform: translateX(-33vw) translateY(39vh) scale(0.56);
  }
  52% {
    transform: translateX(-27vw) translateY(50vh) scale(0.54);
  }
  56% {
    transform: translateX(26vw) translateY(0vh) scale(0.76);
  }
  60% {
    transform: translateX(-7vw) translateY(37vh) scale(0.41);
  }
  64% {
    transform: translateX(36vw) translateY(-44vh) scale(0.81);
  }
  68% {
    transform: translateX(46vw) translateY(-3vh) scale(0.66);
  }
  72% {
    transform: translateX(12vw) translateY(-46vh) scale(0.96);
  }
  76% {
    transform: translateX(23vw) translateY(-8vh) scale(0.73);
  }
  80% {
    transform: translateX(46vw) translateY(28vh) scale(0.47);
  }
  84% {
    transform: translateX(38vw) translateY(27vh) scale(0.4);
  }
  88% {
    transform: translateX(-40vw) translateY(26vh) scale(0.63);
  }
  92% {
    transform: translateX(-26vw) translateY(-44vh) scale(0.48);
  }
  96% {
    transform: translateX(40vw) translateY(28vh) scale(0.69);
  }
  100% {
    transform: translateX(35vw) translateY(41vh) scale(0.6);
  }
}
.firefly:nth-child(3) {
  animation-name: move3;
}
.firefly:nth-child(3)::before {
  animation-duration: 10s;
}
.firefly:nth-child(3)::after {
  animation-duration: 10s, 5064ms;
  animation-delay: 0ms, 6938ms;
}

@keyframes move3 {
  0% {
    transform: translateX(50vw) translateY(-3vh) scale(0.92);
  }
  4.5454545455% {
    transform: translateX(4vw) translateY(-25vh) scale(0.57);
  }
  9.0909090909% {
    transform: translateX(-26vw) translateY(2vh) scale(0.31);
  }
  13.6363636364% {
    transform: translateX(21vw) translateY(-38vh) scale(0.35);
  }
  18.1818181818% {
    transform: translateX(-19vw) translateY(-7vh) scale(0.58);
  }
  22.7272727273% {
    transform: translateX(31vw) translateY(28vh) scale(0.38);
  }
  27.2727272727% {
    transform: translateX(-18vw) translateY(9vh) scale(0.67);
  }
  31.8181818182% {
    transform: translateX(-27vw) translateY(29vh) scale(0.39);
  }
  36.3636363636% {
    transform: translateX(-34vw) translateY(-28vh) scale(0.52);
  }
  40.9090909091% {
    transform: translateX(31vw) translateY(44vh) scale(0.58);
  }
  45.4545454545% {
    transform: translateX(29vw) translateY(34vh) scale(0.94);
  }
  50% {
    transform: translateX(-20vw) translateY(35vh) scale(0.56);
  }
  54.5454545455% {
    transform: translateX(31vw) translateY(17vh) scale(0.51);
  }
  59.0909090909% {
    transform: translateX(-39vw) translateY(-4vh) scale(0.88);
  }
  63.6363636364% {
    transform: translateX(-19vw) translateY(34vh) scale(0.43);
  }
  68.1818181818% {
    transform: translateX(50vw) translateY(38vh) scale(0.58);
  }
  72.7272727273% {
    transform: translateX(-13vw) translateY(-28vh) scale(0.35);
  }
  77.2727272727% {
    transform: translateX(-44vw) translateY(-12vh) scale(0.8);
  }
  81.8181818182% {
    transform: translateX(44vw) translateY(-3vh) scale(0.83);
  }
  86.3636363636% {
    transform: translateX(18vw) translateY(-34vh) scale(0.98);
  }
  90.9090909091% {
    transform: translateX(38vw) translateY(-3vh) scale(0.97);
  }
  95.4545454545% {
    transform: translateX(-2vw) translateY(-28vh) scale(0.96);
  }
  100% {
    transform: translateX(16vw) translateY(-43vh) scale(0.91);
  }
}
.firefly:nth-child(4) {
  animation-name: move4;
}
.firefly:nth-child(4)::before {
  animation-duration: 12s;
}
.firefly:nth-child(4)::after {
  animation-duration: 12s, 5227ms;
  animation-delay: 0ms, 2013ms;
}

@keyframes move4 {
  0% {
    transform: translateX(21vw) translateY(11vh) scale(0.56);
  }
  3.8461538462% {
    transform: translateX(36vw) translateY(-41vh) scale(0.6);
  }
  7.6923076923% {
    transform: translateX(39vw) translateY(4vh) scale(0.37);
  }
  11.5384615385% {
    transform: translateX(-36vw) translateY(45vh) scale(0.66);
  }
  15.3846153846% {
    transform: translateX(-30vw) translateY(-42vh) scale(0.43);
  }
  19.2307692308% {
    transform: translateX(-23vw) translateY(-7vh) scale(0.85);
  }
  23.0769230769% {
    transform: translateX(-6vw) translateY(-40vh) scale(0.6);
  }
  26.9230769231% {
    transform: translateX(7vw) translateY(11vh) scale(0.73);
  }
  30.7692307692% {
    transform: translateX(31vw) translateY(48vh) scale(0.81);
  }
  34.6153846154% {
    transform: translateX(-22vw) translateY(5vh) scale(0.92);
  }
  38.4615384615% {
    transform: translateX(38vw) translateY(-10vh) scale(0.86);
  }
  42.3076923077% {
    transform: translateX(16vw) translateY(37vh) scale(0.5);
  }
  46.1538461538% {
    transform: translateX(-14vw) translateY(-39vh) scale(0.35);
  }
  50% {
    transform: translateX(42vw) translateY(-47vh) scale(0.32);
  }
  53.8461538462% {
    transform: translateX(7vw) translateY(41vh) scale(0.77);
  }
  57.6923076923% {
    transform: translateX(-46vw) translateY(-19vh) scale(0.38);
  }
  61.5384615385% {
    transform: translateX(1vw) translateY(-29vh) scale(0.66);
  }
  65.3846153846% {
    transform: translateX(-13vw) translateY(22vh) scale(0.66);
  }
  69.2307692308% {
    transform: translateX(-28vw) translateY(39vh) scale(0.91);
  }
  73.0769230769% {
    transform: translateX(13vw) translateY(27vh) scale(0.55);
  }
  76.9230769231% {
    transform: translateX(-37vw) translateY(15vh) scale(0.89);
  }
  80.7692307692% {
    transform: translateX(9vw) translateY(-4vh) scale(0.72);
  }
  84.6153846154% {
    transform: translateX(27vw) translateY(-45vh) scale(0.46);
  }
  88.4615384615% {
    transform: translateX(28vw) translateY(-34vh) scale(0.73);
  }
  92.3076923077% {
    transform: translateX(23vw) translateY(-26vh) scale(0.63);
  }
  96.1538461538% {
    transform: translateX(-12vw) translateY(-33vh) scale(0.46);
  }
  100% {
    transform: translateX(34vw) translateY(30vh) scale(0.73);
  }
}
.firefly:nth-child(5) {
  animation-name: move5;
}
.firefly:nth-child(5)::before {
  animation-duration: 11s;
}
.firefly:nth-child(5)::after {
  animation-duration: 11s, 7047ms;
  animation-delay: 0ms, 5251ms;
}

@keyframes move5 {
  0% {
    transform: translateX(-24vw) translateY(-15vh) scale(0.96);
  }
  4.7619047619% {
    transform: translateX(4vw) translateY(-2vh) scale(0.84);
  }
  9.5238095238% {
    transform: translateX(28vw) translateY(2vh) scale(0.83);
  }
  14.2857142857% {
    transform: translateX(-47vw) translateY(33vh) scale(0.4);
  }
  19.0476190476% {
    transform: translateX(-7vw) translateY(44vh) scale(0.77);
  }
  23.8095238095% {
    transform: translateX(-25vw) translateY(-31vh) scale(0.98);
  }
  28.5714285714% {
    transform: translateX(-13vw) translateY(36vh) scale(0.56);
  }
  33.3333333333% {
    transform: translateX(-34vw) translateY(17vh) scale(0.89);
  }
  38.0952380952% {
    transform: translateX(-43vw) translateY(1vh) scale(0.8);
  }
  42.8571428571% {
    transform: translateX(-41vw) translateY(17vh) scale(0.34);
  }
  47.619047619% {
    transform: translateX(48vw) translateY(4vh) scale(0.76);
  }
  52.380952381% {
    transform: translateX(10vw) translateY(23vh) scale(0.74);
  }
  57.1428571429% {
    transform: translateX(-45vw) translateY(45vh) scale(0.42);
  }
  61.9047619048% {
    transform: translateX(-5vw) translateY(-19vh) scale(0.93);
  }
  66.6666666667% {
    transform: translateX(-44vw) translateY(-37vh) scale(0.72);
  }
  71.4285714286% {
    transform: translateX(5vw) translateY(32vh) scale(0.79);
  }
  76.1904761905% {
    transform: translateX(-32vw) translateY(-31vh) scale(0.36);
  }
  80.9523809524% {
    transform: translateX(-18vw) translateY(-7vh) scale(0.94);
  }
  85.7142857143% {
    transform: translateX(-46vw) translateY(-49vh) scale(0.89);
  }
  90.4761904762% {
    transform: translateX(25vw) translateY(7vh) scale(0.87);
  }
  95.2380952381% {
    transform: translateX(25vw) translateY(-33vh) scale(0.4);
  }
  100% {
    transform: translateX(48vw) translateY(-23vh) scale(0.96);
  }
}
.firefly:nth-child(6) {
  animation-name: move6;
}
.firefly:nth-child(6)::before {
  animation-duration: 17s;
}
.firefly:nth-child(6)::after {
  animation-duration: 17s, 8139ms;
  animation-delay: 0ms, 1462ms;
}

@keyframes move6 {
  0% {
    transform: translateX(50vw) translateY(4vh) scale(0.43);
  }
  5.5555555556% {
    transform: translateX(43vw) translateY(23vh) scale(0.7);
  }
  11.1111111111% {
    transform: translateX(-46vw) translateY(-44vh) scale(0.72);
  }
  16.6666666667% {
    transform: translateX(43vw) translateY(-42vh) scale(0.38);
  }
  22.2222222222% {
    transform: translateX(-26vw) translateY(15vh) scale(0.77);
  }
  27.7777777778% {
    transform: translateX(39vw) translateY(-15vh) scale(0.93);
  }
  33.3333333333% {
    transform: translateX(-41vw) translateY(28vh) scale(0.67);
  }
  38.8888888889% {
    transform: translateX(-14vw) translateY(-13vh) scale(0.49);
  }
  44.4444444444% {
    transform: translateX(-38vw) translateY(21vh) scale(0.43);
  }
  50% {
    transform: translateX(-45vw) translateY(-39vh) scale(0.49);
  }
  55.5555555556% {
    transform: translateX(27vw) translateY(12vh) scale(0.96);
  }
  61.1111111111% {
    transform: translateX(42vw) translateY(12vh) scale(0.59);
  }
  66.6666666667% {
    transform: translateX(4vw) translateY(-8vh) scale(0.7);
  }
  72.2222222222% {
    transform: translateX(6vw) translateY(-48vh) scale(0.8);
  }
  77.7777777778% {
    transform: translateX(-31vw) translateY(46vh) scale(0.43);
  }
  83.3333333333% {
    transform: translateX(-9vw) translateY(28vh) scale(0.42);
  }
  88.8888888889% {
    transform: translateX(4vw) translateY(-20vh) scale(0.62);
  }
  94.4444444444% {
    transform: translateX(-12vw) translateY(21vh) scale(0.82);
  }
  100% {
    transform: translateX(34vw) translateY(36vh) scale(0.29);
  }
}
.firefly:nth-child(7) {
  animation-name: move7;
}
.firefly:nth-child(7)::before {
  animation-duration: 13s;
}
.firefly:nth-child(7)::after {
  animation-duration: 13s, 6111ms;
  animation-delay: 0ms, 7139ms;
}

@keyframes move7 {
  0% {
    transform: translateX(22vw) translateY(31vh) scale(0.34);
  }
  4.5454545455% {
    transform: translateX(9vw) translateY(0vh) scale(0.67);
  }
  9.0909090909% {
    transform: translateX(18vw) translateY(-39vh) scale(0.27);
  }
  13.6363636364% {
    transform: translateX(41vw) translateY(-16vh) scale(0.72);
  }
  18.1818181818% {
    transform: translateX(38vw) translateY(8vh) scale(0.27);
  }
  22.7272727273% {
    transform: translateX(-29vw) translateY(-4vh) scale(0.86);
  }
  27.2727272727% {
    transform: translateX(-19vw) translateY(-15vh) scale(0.76);
  }
  31.8181818182% {
    transform: translateX(38vw) translateY(-38vh) scale(0.39);
  }
  36.3636363636% {
    transform: translateX(40vw) translateY(-20vh) scale(0.36);
  }
  40.9090909091% {
    transform: translateX(-47vw) translateY(-45vh) scale(0.68);
  }
  45.4545454545% {
    transform: translateX(-33vw) translateY(34vh) scale(0.92);
  }
  50% {
    transform: translateX(-39vw) translateY(47vh) scale(0.65);
  }
  54.5454545455% {
    transform: translateX(0vw) translateY(-10vh) scale(0.85);
  }
  59.0909090909% {
    transform: translateX(-43vw) translateY(47vh) scale(0.94);
  }
  63.6363636364% {
    transform: translateX(-41vw) translateY(-10vh) scale(0.48);
  }
  68.1818181818% {
    transform: translateX(44vw) translateY(47vh) scale(0.4);
  }
  72.7272727273% {
    transform: translateX(-38vw) translateY(-44vh) scale(0.67);
  }
  77.2727272727% {
    transform: translateX(45vw) translateY(43vh) scale(0.35);
  }
  81.8181818182% {
    transform: translateX(-40vw) translateY(-23vh) scale(0.81);
  }
  86.3636363636% {
    transform: translateX(13vw) translateY(-14vh) scale(0.54);
  }
  90.9090909091% {
    transform: translateX(24vw) translateY(-26vh) scale(0.88);
  }
  95.4545454545% {
    transform: translateX(21vw) translateY(0vh) scale(0.28);
  }
  100% {
    transform: translateX(46vw) translateY(42vh) scale(0.68);
  }
}
.firefly:nth-child(8) {
  animation-name: move8;
}
.firefly:nth-child(8)::before {
  animation-duration: 15s;
}
.firefly:nth-child(8)::after {
  animation-duration: 15s, 9135ms;
  animation-delay: 0ms, 5692ms;
}

@keyframes move8 {
  0% {
    transform: translateX(35vw) translateY(9vh) scale(0.66);
  }
  5% {
    transform: translateX(0vw) translateY(27vh) scale(0.54);
  }
  10% {
    transform: translateX(-38vw) translateY(44vh) scale(0.41);
  }
  15% {
    transform: translateX(30vw) translateY(-17vh) scale(0.68);
  }
  20% {
    transform: translateX(-48vw) translateY(40vh) scale(0.71);
  }
  25% {
    transform: translateX(-9vw) translateY(36vh) scale(0.9);
  }
  30% {
    transform: translateX(20vw) translateY(-32vh) scale(0.78);
  }
  35% {
    transform: translateX(25vw) translateY(-6vh) scale(0.53);
  }
  40% {
    transform: translateX(-13vw) translateY(0vh) scale(0.54);
  }
  45% {
    transform: translateX(16vw) translateY(-43vh) scale(0.32);
  }
  50% {
    transform: translateX(-13vw) translateY(-34vh) scale(0.89);
  }
  55% {
    transform: translateX(8vw) translateY(23vh) scale(0.99);
  }
  60% {
    transform: translateX(-24vw) translateY(49vh) scale(0.51);
  }
  65% {
    transform: translateX(13vw) translateY(-15vh) scale(0.79);
  }
  70% {
    transform: translateX(0vw) translateY(-18vh) scale(0.56);
  }
  75% {
    transform: translateX(48vw) translateY(45vh) scale(0.92);
  }
  80% {
    transform: translateX(-41vw) translateY(-47vh) scale(0.41);
  }
  85% {
    transform: translateX(36vw) translateY(49vh) scale(0.97);
  }
  90% {
    transform: translateX(25vw) translateY(7vh) scale(0.8);
  }
  95% {
    transform: translateX(-41vw) translateY(-30vh) scale(0.65);
  }
  100% {
    transform: translateX(15vw) translateY(-7vh) scale(0.39);
  }
}
.firefly:nth-child(9) {
  animation-name: move9;
}
.firefly:nth-child(9)::before {
  animation-duration: 17s;
}
.firefly:nth-child(9)::after {
  animation-duration: 17s, 6403ms;
  animation-delay: 0ms, 3411ms;
}

@keyframes move9 {
  0% {
    transform: translateX(-8vw) translateY(-21vh) scale(0.41);
  }
  4% {
    transform: translateX(-33vw) translateY(-46vh) scale(0.57);
  }
  8% {
    transform: translateX(45vw) translateY(-45vh) scale(0.3);
  }
  12% {
    transform: translateX(17vw) translateY(16vh) scale(0.77);
  }
  16% {
    transform: translateX(13vw) translateY(-21vh) scale(0.72);
  }
  20% {
    transform: translateX(-39vw) translateY(8vh) scale(0.26);
  }
  24% {
    transform: translateX(-15vw) translateY(47vh) scale(0.5);
  }
  28% {
    transform: translateX(-9vw) translateY(30vh) scale(0.94);
  }
  32% {
    transform: translateX(-38vw) translateY(3vh) scale(0.75);
  }
  36% {
    transform: translateX(17vw) translateY(-21vh) scale(0.48);
  }
  40% {
    transform: translateX(-12vw) translateY(-20vh) scale(0.49);
  }
  44% {
    transform: translateX(37vw) translateY(25vh) scale(0.72);
  }
  48% {
    transform: translateX(-27vw) translateY(14vh) scale(0.73);
  }
  52% {
    transform: translateX(-23vw) translateY(38vh) scale(0.53);
  }
  56% {
    transform: translateX(-26vw) translateY(20vh) scale(0.53);
  }
  60% {
    transform: translateX(-21vw) translateY(-4vh) scale(0.53);
  }
  64% {
    transform: translateX(-43vw) translateY(-15vh) scale(0.52);
  }
  68% {
    transform: translateX(-44vw) translateY(-39vh) scale(0.63);
  }
  72% {
    transform: translateX(-17vw) translateY(-47vh) scale(0.4);
  }
  76% {
    transform: translateX(47vw) translateY(-5vh) scale(0.63);
  }
  80% {
    transform: translateX(-2vw) translateY(-5vh) scale(0.8);
  }
  84% {
    transform: translateX(33vw) translateY(31vh) scale(0.55);
  }
  88% {
    transform: translateX(6vw) translateY(32vh) scale(0.46);
  }
  92% {
    transform: translateX(-8vw) translateY(24vh) scale(0.63);
  }
  96% {
    transform: translateX(-29vw) translateY(-20vh) scale(0.96);
  }
  100% {
    transform: translateX(18vw) translateY(48vh) scale(0.26);
  }
}
.firefly:nth-child(10) {
  animation-name: move10;
}
.firefly:nth-child(10)::before {
  animation-duration: 11s;
}
.firefly:nth-child(10)::after {
  animation-duration: 11s, 9150ms;
  animation-delay: 0ms, 2799ms;
}

@keyframes move10 {
  0% {
    transform: translateX(-22vw) translateY(31vh) scale(0.34);
  }
  4.5454545455% {
    transform: translateX(25vw) translateY(4vh) scale(0.78);
  }
  9.0909090909% {
    transform: translateX(18vw) translateY(20vh) scale(0.34);
  }
  13.6363636364% {
    transform: translateX(44vw) translateY(-47vh) scale(0.5);
  }
  18.1818181818% {
    transform: translateX(-10vw) translateY(27vh) scale(0.47);
  }
  22.7272727273% {
    transform: translateX(4vw) translateY(-47vh) scale(0.64);
  }
  27.2727272727% {
    transform: translateX(12vw) translateY(-6vh) scale(0.76);
  }
  31.8181818182% {
    transform: translateX(4vw) translateY(-48vh) scale(0.87);
  }
  36.3636363636% {
    transform: translateX(-26vw) translateY(-2vh) scale(0.9);
  }
  40.9090909091% {
    transform: translateX(18vw) translateY(17vh) scale(0.36);
  }
  45.4545454545% {
    transform: translateX(-2vw) translateY(48vh) scale(0.42);
  }
  50% {
    transform: translateX(38vw) translateY(40vh) scale(0.38);
  }
  54.5454545455% {
    transform: translateX(-45vw) translateY(-5vh) scale(0.57);
  }
  59.0909090909% {
    transform: translateX(13vw) translateY(35vh) scale(0.39);
  }
  63.6363636364% {
    transform: translateX(-34vw) translateY(37vh) scale(0.34);
  }
  68.1818181818% {
    transform: translateX(-15vw) translateY(12vh) scale(0.87);
  }
  72.7272727273% {
    transform: translateX(-8vw) translateY(22vh) scale(0.79);
  }
  77.2727272727% {
    transform: translateX(1vw) translateY(-14vh) scale(0.29);
  }
  81.8181818182% {
    transform: translateX(-26vw) translateY(-6vh) scale(0.93);
  }
  86.3636363636% {
    transform: translateX(45vw) translateY(39vh) scale(0.33);
  }
  90.9090909091% {
    transform: translateX(-26vw) translateY(-19vh) scale(0.53);
  }
  95.4545454545% {
    transform: translateX(-42vw) translateY(39vh) scale(0.65);
  }
  100% {
    transform: translateX(21vw) translateY(-37vh) scale(0.77);
  }
}
.firefly:nth-child(11) {
  animation-name: move11;
}
.firefly:nth-child(11)::before {
  animation-duration: 11s;
}
.firefly:nth-child(11)::after {
  animation-duration: 11s, 8392ms;
  animation-delay: 0ms, 7276ms;
}

@keyframes move11 {
  0% {
    transform: translateX(-7vw) translateY(39vh) scale(0.88);
  }
  5.5555555556% {
    transform: translateX(-4vw) translateY(-13vh) scale(0.39);
  }
  11.1111111111% {
    transform: translateX(-1vw) translateY(25vh) scale(0.8);
  }
  16.6666666667% {
    transform: translateX(0vw) translateY(30vh) scale(0.5);
  }
  22.2222222222% {
    transform: translateX(11vw) translateY(-9vh) scale(0.28);
  }
  27.7777777778% {
    transform: translateX(19vw) translateY(-9vh) scale(0.36);
  }
  33.3333333333% {
    transform: translateX(-32vw) translateY(28vh) scale(0.87);
  }
  38.8888888889% {
    transform: translateX(39vw) translateY(-46vh) scale(0.9);
  }
  44.4444444444% {
    transform: translateX(40vw) translateY(-18vh) scale(0.98);
  }
  50% {
    transform: translateX(18vw) translateY(-19vh) scale(0.76);
  }
  55.5555555556% {
    transform: translateX(48vw) translateY(-38vh) scale(0.79);
  }
  61.1111111111% {
    transform: translateX(-47vw) translateY(-2vh) scale(0.95);
  }
  66.6666666667% {
    transform: translateX(21vw) translateY(-39vh) scale(0.44);
  }
  72.2222222222% {
    transform: translateX(28vw) translateY(42vh) scale(0.39);
  }
  77.7777777778% {
    transform: translateX(-30vw) translateY(17vh) scale(0.9);
  }
  83.3333333333% {
    transform: translateX(-25vw) translateY(-37vh) scale(0.49);
  }
  88.8888888889% {
    transform: translateX(-22vw) translateY(-6vh) scale(0.77);
  }
  94.4444444444% {
    transform: translateX(-28vw) translateY(35vh) scale(0.75);
  }
  100% {
    transform: translateX(-13vw) translateY(-28vh) scale(0.77);
  }
}
.firefly:nth-child(12) {
  animation-name: move12;
}
.firefly:nth-child(12)::before {
  animation-duration: 15s;
}
.firefly:nth-child(12)::after {
  animation-duration: 15s, 10809ms;
  animation-delay: 0ms, 1032ms;
}

@keyframes move12 {
  0% {
    transform: translateX(14vw) translateY(-14vh) scale(0.62);
  }
  4.1666666667% {
    transform: translateX(-33vw) translateY(4vh) scale(0.5);
  }
  8.3333333333% {
    transform: translateX(-12vw) translateY(17vh) scale(0.94);
  }
  12.5% {
    transform: translateX(50vw) translateY(-16vh) scale(0.56);
  }
  16.6666666667% {
    transform: translateX(8vw) translateY(39vh) scale(0.34);
  }
  20.8333333333% {
    transform: translateX(21vw) translateY(41vh) scale(0.48);
  }
  25% {
    transform: translateX(15vw) translateY(-32vh) scale(0.6);
  }
  29.1666666667% {
    transform: translateX(-22vw) translateY(44vh) scale(0.59);
  }
  33.3333333333% {
    transform: translateX(-22vw) translateY(-42vh) scale(0.95);
  }
  37.5% {
    transform: translateX(-30vw) translateY(-9vh) scale(0.64);
  }
  41.6666666667% {
    transform: translateX(17vw) translateY(-3vh) scale(0.77);
  }
  45.8333333333% {
    transform: translateX(34vw) translateY(-25vh) scale(0.69);
  }
  50% {
    transform: translateX(32vw) translateY(21vh) scale(0.98);
  }
  54.1666666667% {
    transform: translateX(-16vw) translateY(42vh) scale(0.68);
  }
  58.3333333333% {
    transform: translateX(7vw) translateY(12vh) scale(0.87);
  }
  62.5% {
    transform: translateX(-7vw) translateY(41vh) scale(0.85);
  }
  66.6666666667% {
    transform: translateX(33vw) translateY(-29vh) scale(0.84);
  }
  70.8333333333% {
    transform: translateX(50vw) translateY(4vh) scale(0.94);
  }
  75% {
    transform: translateX(13vw) translateY(-11vh) scale(0.59);
  }
  79.1666666667% {
    transform: translateX(-40vw) translateY(-21vh) scale(0.96);
  }
  83.3333333333% {
    transform: translateX(49vw) translateY(-6vh) scale(0.76);
  }
  87.5% {
    transform: translateX(21vw) translateY(3vh) scale(0.52);
  }
  91.6666666667% {
    transform: translateX(38vw) translateY(-33vh) scale(0.99);
  }
  95.8333333333% {
    transform: translateX(11vw) translateY(-11vh) scale(0.31);
  }
  100% {
    transform: translateX(-43vw) translateY(46vh) scale(0.33);
  }
}
.firefly:nth-child(13) {
  animation-name: move13;
}
.firefly:nth-child(13)::before {
  animation-duration: 17s;
}
.firefly:nth-child(13)::after {
  animation-duration: 17s, 5267ms;
  animation-delay: 0ms, 2127ms;
}

@keyframes move13 {
  0% {
    transform: translateX(22vw) translateY(14vh) scale(0.69);
  }
  5.5555555556% {
    transform: translateX(46vw) translateY(-22vh) scale(0.83);
  }
  11.1111111111% {
    transform: translateX(-21vw) translateY(-17vh) scale(0.39);
  }
  16.6666666667% {
    transform: translateX(-26vw) translateY(-22vh) scale(0.98);
  }
  22.2222222222% {
    transform: translateX(-39vw) translateY(-26vh) scale(0.39);
  }
  27.7777777778% {
    transform: translateX(-6vw) translateY(17vh) scale(0.42);
  }
  33.3333333333% {
    transform: translateX(-26vw) translateY(-25vh) scale(0.54);
  }
  38.8888888889% {
    transform: translateX(-39vw) translateY(-11vh) scale(0.69);
  }
  44.4444444444% {
    transform: translateX(6vw) translateY(18vh) scale(0.45);
  }
  50% {
    transform: translateX(-9vw) translateY(-28vh) scale(0.35);
  }
  55.5555555556% {
    transform: translateX(21vw) translateY(-6vh) scale(0.57);
  }
  61.1111111111% {
    transform: translateX(-41vw) translateY(19vh) scale(0.72);
  }
  66.6666666667% {
    transform: translateX(22vw) translateY(-42vh) scale(0.49);
  }
  72.2222222222% {
    transform: translateX(-48vw) translateY(-36vh) scale(0.42);
  }
  77.7777777778% {
    transform: translateX(4vw) translateY(15vh) scale(0.62);
  }
  83.3333333333% {
    transform: translateX(-3vw) translateY(9vh) scale(0.47);
  }
  88.8888888889% {
    transform: translateX(-7vw) translateY(-4vh) scale(0.36);
  }
  94.4444444444% {
    transform: translateX(-14vw) translateY(34vh) scale(0.84);
  }
  100% {
    transform: translateX(-27vw) translateY(37vh) scale(0.53);
  }
}
.firefly:nth-child(14) {
  animation-name: move14;
}
.firefly:nth-child(14)::before {
  animation-duration: 9s;
}
.firefly:nth-child(14)::after {
  animation-duration: 9s, 9870ms;
  animation-delay: 0ms, 575ms;
}

@keyframes move14 {
  0% {
    transform: translateX(-45vw) translateY(27vh) scale(0.94);
  }
  5% {
    transform: translateX(-2vw) translateY(21vh) scale(0.42);
  }
  10% {
    transform: translateX(-17vw) translateY(27vh) scale(0.5);
  }
  15% {
    transform: translateX(-47vw) translateY(1vh) scale(0.72);
  }
  20% {
    transform: translateX(45vw) translateY(-18vh) scale(0.66);
  }
  25% {
    transform: translateX(28vw) translateY(-10vh) scale(0.53);
  }
  30% {
    transform: translateX(14vw) translateY(9vh) scale(0.72);
  }
  35% {
    transform: translateX(22vw) translateY(13vh) scale(0.76);
  }
  40% {
    transform: translateX(44vw) translateY(-27vh) scale(0.66);
  }
  45% {
    transform: translateX(-43vw) translateY(-9vh) scale(0.69);
  }
  50% {
    transform: translateX(-1vw) translateY(13vh) scale(0.43);
  }
  55% {
    transform: translateX(25vw) translateY(47vh) scale(0.27);
  }
  60% {
    transform: translateX(-25vw) translateY(37vh) scale(0.73);
  }
  65% {
    transform: translateX(47vw) translateY(-38vh) scale(0.26);
  }
  70% {
    transform: translateX(-23vw) translateY(-20vh) scale(0.48);
  }
  75% {
    transform: translateX(27vw) translateY(-14vh) scale(0.78);
  }
  80% {
    transform: translateX(-49vw) translateY(47vh) scale(0.56);
  }
  85% {
    transform: translateX(-10vw) translateY(46vh) scale(0.3);
  }
  90% {
    transform: translateX(26vw) translateY(-26vh) scale(0.67);
  }
  95% {
    transform: translateX(44vw) translateY(42vh) scale(0.85);
  }
  100% {
    transform: translateX(3vw) translateY(42vh) scale(1);
  }
}
.firefly:nth-child(15) {
  animation-name: move15;
}
.firefly:nth-child(15)::before {
  animation-duration: 10s;
}
.firefly:nth-child(15)::after {
  animation-duration: 10s, 9204ms;
  animation-delay: 0ms, 5861ms;
}

@keyframes move15 {
  0% {
    transform: translateX(41vw) translateY(21vh) scale(0.99);
  }
  4.1666666667% {
    transform: translateX(-37vw) translateY(4vh) scale(0.46);
  }
  8.3333333333% {
    transform: translateX(36vw) translateY(-6vh) scale(0.61);
  }
  12.5% {
    transform: translateX(-39vw) translateY(29vh) scale(0.27);
  }
  16.6666666667% {
    transform: translateX(42vw) translateY(2vh) scale(0.88);
  }
  20.8333333333% {
    transform: translateX(38vw) translateY(31vh) scale(0.46);
  }
  25% {
    transform: translateX(49vw) translateY(-22vh) scale(0.58);
  }
  29.1666666667% {
    transform: translateX(12vw) translateY(24vh) scale(0.31);
  }
  33.3333333333% {
    transform: translateX(-25vw) translateY(-35vh) scale(0.49);
  }
  37.5% {
    transform: translateX(-17vw) translateY(-38vh) scale(0.34);
  }
  41.6666666667% {
    transform: translateX(-15vw) translateY(5vh) scale(0.96);
  }
  45.8333333333% {
    transform: translateX(-37vw) translateY(30vh) scale(0.74);
  }
  50% {
    transform: translateX(31vw) translateY(31vh) scale(0.76);
  }
  54.1666666667% {
    transform: translateX(-15vw) translateY(40vh) scale(0.35);
  }
  58.3333333333% {
    transform: translateX(49vw) translateY(-34vh) scale(0.98);
  }
  62.5% {
    transform: translateX(31vw) translateY(46vh) scale(0.92);
  }
  66.6666666667% {
    transform: translateX(-8vw) translateY(-8vh) scale(0.38);
  }
  70.8333333333% {
    transform: translateX(9vw) translateY(-35vh) scale(0.72);
  }
  75% {
    transform: translateX(-3vw) translateY(-22vh) scale(0.57);
  }
  79.1666666667% {
    transform: translateX(32vw) translateY(4vh) scale(0.89);
  }
  83.3333333333% {
    transform: translateX(-17vw) translateY(22vh) scale(0.42);
  }
  87.5% {
    transform: translateX(22vw) translateY(20vh) scale(0.77);
  }
  91.6666666667% {
    transform: translateX(12vw) translateY(45vh) scale(0.61);
  }
  95.8333333333% {
    transform: translateX(4vw) translateY(32vh) scale(0.87);
  }
  100% {
    transform: translateX(15vw) translateY(-24vh) scale(0.57);
  }
}
@keyframes drift {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
@keyframes flash {
  0%,
  30%,
  100% {
    opacity: 0;
    box-shadow: 0 0 0vw 0vw yellow;
  }
  5% {
    opacity: 1;
    box-shadow: 0 0 2vw 0.4vw yellow;
  }
}
</style>
