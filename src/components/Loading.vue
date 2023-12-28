<template>
  <div class="v3ti-loader-wrapper">
    <div class="v3ti-loader"></div>
    <span>Loading</span>
  </div>
</template>

<style lang="scss">
$loadingSize: 18px;
$loadingColor: #112B3C;
.v3ti-loader-wrapper {
  display: flex;
  align-items: center;
  justify-content: center;
  color: $loadingColor;
  .v3ti-loader {
    width: $loadingSize;
    height: $loadingSize;
    border-radius: 50%;
    display: inline-block;
    border-top: 2px solid $loadingColor;
    border-right: 2px solid transparent;
    box-sizing: border-box;
    animation: rotation .8s linear infinite;
    margin-right: 8px;
  }
  @keyframes rotation {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }
}

</style>