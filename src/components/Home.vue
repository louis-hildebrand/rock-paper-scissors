<template>
  <div class="playing-area" ref="playing_area">
    <div v-for="rock in rocks" :key="rock.index">
      <img v-bind:src="require('@/assets/rock.png')" alt="rock" v-bind:style="img_style(rock.index)" />
    </div>
    <div v-for="paper in papers" :key="paper.index">
      <img v-bind:src="require('@/assets/paper.png')" alt="paper" v-bind:style="img_style(paper.index)" />
    </div>
    <div v-for="scissor in scissors" :key="scissor.index">
      <img v-bind:src="require('@/assets/scissors.png')" alt="scissor" v-bind:style="img_style(scissor.index)" />
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      polling: null,
      playing_area_width: 0,
      playing_area_height: 0,
      img_size: 0,
      num_objects: 60,
      obj_types: [],
      positions: [],
      directions: [],
      speed_x: 150,
      speed_y: 150,
      refresh_time: 33
    }
  },
  computed: {
    rocks: function() {
      return this.objects_by_type("rock");
    },
    papers: function() {
      return this.objects_by_type("paper");
    },
    scissors: function() {
      return this.objects_by_type("scissors");
    }
  },
  methods: {
    init() {
      const playing_area_rect = this.$refs.playing_area.getBoundingClientRect();
      this.playing_area_width = playing_area_rect.width;
      this.playing_area_height = playing_area_rect.height;
      this.img_size = 0.1 * Math.min(this.playing_area_width, this.playing_area_height)

      this.generate_initial_objects()
      
      this.polling = setInterval(this.move_all, this.refresh_time);
    },
    generate_initial_objects() {
      for (var i = 0; i < this.num_objects; i++) {
        const r = Math.random();
        const obj_type =
          r < 1/3 ? "rock" :
          r < 2/3 ? "paper" :
          "scissors";
        this.obj_types[i] = obj_type;

        const pos_x = Math.random() * (this.playing_area_width - this.img_size);
        const pos_y = Math.random() * (this.playing_area_height - this.img_size);
        this.positions[i] = {x: pos_x, y: pos_y};
        
        const dir_x = Math.random() < 0.5 ? -1 : 1;
        const dir_y = Math.random() < 0.5 ? -1 : 1;
        this.directions[i] = {x: dir_x, y: dir_y};
      }
    },
    vel_x(index) {
      return this.speed_x * this.directions[index].x;
    },
    vel_y(index) {
      return this.speed_y * this.directions[index].y;
    },
    objects_by_type(target_type) {
      var objs = []
      for (var i = 0; i < this.num_objects; i++) {
        if (this.obj_types[i] === target_type) {
          objs.push({index:i, position:this.positions[i], direction:this.directions[i]});
        }
      }
      return objs;
    },
    img_style(index) {
      const pos_x = this.positions[index].x;
      const pos_y = this.positions[index].y;
      return "position:absolute; left:" + pos_x + "px; top:" + pos_y + "px; width:" + this.img_size + "px; height: " + this.img_size + "px;";
    },
    are_colliding(i, j) {
      const pos_i = this.positions[i];
      const pos_j = this.positions[j];
      const intersecting = Math.abs(pos_i.x - pos_j.x) < this.img_size && Math.abs(pos_i.y - pos_j.y) < this.img_size;
      
      var moving_closer = false;
      const [left, right] = pos_i.x < pos_j.x ? [i, j] : [j, i];
      if (this.directions[left].x == 1 && this.directions[right].x == -1)
        moving_closer = true;
      const [top, bot] = pos_i.y < pos_j.y ? [i, j] : [j, i];
      if (this.directions[top].y == 1 && this.directions[bot].y == -1)
        moving_closer = true;

      return intersecting && moving_closer;
    },
    play_rock_paper_scissors(i, j) {
      const o_types = [this.obj_types[i], this.obj_types[j]];
      if (o_types.includes("rock") && o_types.includes("paper")) {
        this.obj_types[i] = "paper"
        this.obj_types[j] = "paper"
      }
      else if (o_types.includes("rock") && o_types.includes("scissors")) {
        this.obj_types[i] = "rock"
        this.obj_types[j] = "rock"
      }
      else if (o_types.includes("paper") && o_types.includes("scissors")) {
        this.obj_types[i] = "scissors"
        this.obj_types[j] = "scissors"
      }
      else {
        return;
      }
    },
    update_colliding_objects(i, j) {
      this.play_rock_paper_scissors(i, j);
      const temp = this.directions[i];
      this.directions[i] = this.directions[j];
      this.directions[j] = temp;
    },
    all_same_type() {
      const first_type = this.obj_types[0];
      for (var i = 1; i < this.num_objects; i++) {
        if (this.obj_types[i] !== first_type)
          return false;
      }
      return true;
    },
    move_all() {
      for (var i = 0; i < this.num_objects; i++) {
        this.move(i);
      }
      
      for (var i = 0; i < this.num_objects - 1; i++) {
        for (var j = i + 1; j < this.num_objects; j++) {
          if (this.are_colliding(i, j)) {
            this.update_colliding_objects(i, j)
          }
        }
      }

      if (this.all_same_type())
        clearInterval(this.polling)
    },
    move(index) {
      this.positions[index].x += this.vel_x(index) * this.refresh_time / 1000;
      if (this.positions[index].x <= 0) {
        this.directions[index].x = 1;
        this.positions[index].x = 0;
      }
      else if (this.positions[index].x + this.img_size >= this.playing_area_width) {
        this.directions[index].x = -1;
        this.positions[index].x = this.playing_area_width - this.img_size;
      }
      
      this.positions[index].y += this.vel_y(index) * this.refresh_time / 1000;
      if (this.positions[index].y <= 0) {
        this.directions[index].y = 1;
        this.positions[index].y = 0;
      }
      else if (this.positions[index].y + this.img_size >= this.playing_area_height) {
        this.directions[index].y = -1;
        this.positions[index].y = this.playing_area_height - this.img_size;
      }
    }
  },
  mounted() {
    this.init();
  }
}
</script>

<style>
.playing-area {
  position:absolute;
  left:0vw;
  top:0vh;
  width:100vw;
  height:100vh;
}
</style>
