<template>
  <div class="StyledButton-btn">
    <slot/>
  </div>
</template>
<style scoped>
.StyledButton-btn {
  padding-inline: 20px;
  padding-block: 5px;
  text-align: center;
  border-radius: 15px;
  background-color: white;
  color: black;
}
.StyledButton-btn:hover {
  cursor: pointer;
}
</style>