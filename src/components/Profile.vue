<template>
  <form class="form-widget" @submit.prevent="updateProfile">
    <Avatar v-model:path="avatar_url" @upload="updateProfile" />
    <div>
      <label for="email">Email</label>
      <input id="email" type="text" :value="store.user.email" disabled />
    </div>
    <div>
      <label for="username">Name</label>
      <input id="username" type="text" v-model="username" />
    </div>
    <div>
      <label for="website">Website</label>
      <input id="website" type="website" v-model="website" />
    </div>

    <div>
      <input
        type="submit"
        class="button primary block"
        :value="loading ? 'Loading ...' : 'Update'"
        :disabled="loading"
      />
    </div>

    <div>
      <button class="button block" @click="signOut" :disabled="loading">
        Sign Out
      </button>
    </div>
  </form>
</template>

<script>
import { supabase } from "../supabase";
import { store } from "../store";
import { onMounted, ref } from "vue";
import Avatar from "./Avatar.vue";
import * as THREE from "three";

export default {
  setup() {
    const loading = ref(true);
    const username = ref("");
    const website = ref("");
    const avatar_url = ref("");

    async function getProfile() {
      try {
        loading.value = true;
        store.user = supabase.auth.user();

        let { data, error, status } = await supabase
          .from("profiles")
          .select(`username, website, avatar_url`)
          .eq("id", store.user.id)
          .single();

        if (error && status !== 406) throw error;

        if (data) {
          username.value = data.username;
          website.value = data.website;
          avatar_url.value = data.avatar_url;
        }
      } catch (error) {
        alert(error.message);
      } finally {
        loading.value = false;
      }
    }

    async function updateProfile() {
      try {
        loading.value = true;
        store.user = supabase.auth.user();

        const updates = {
          id: store.user.id,
          username: username.value,
          website: website.value,
          avatar_url: avatar_url.value,
          updated_at: new Date(),
        };

        let { error } = await supabase.from("profiles").upsert(updates, {
          returning: "minimal", // Don't return the value after inserting
        });

        if (error) throw error;
      } catch (error) {
        alert(error.message);
      } finally {
        loading.value = false;
      }
    }

    async function signOut() {
      try {
        loading.value = true;
        let { error } = await supabase.auth.signOut();
        if (error) throw error;
      } catch (error) {
        alert(error.message);
      } finally {
        loading.value = false;
      }
    }

    onMounted(() => {
      getProfile();

      const scene = new THREE.Scene();
      const camera = new THREE.PerspectiveCamera(
        75,
        window.innerWidth / window.innerHeight,
        0.1,
        1000
      );
      const renderer = new THREE.WebGLRenderer();
      renderer.setSize(window.innerWidth, window.innerHeight);

      document.body.appendChild(renderer.domElement);

      const geometry = new THREE.BoxGeometry(1, 1, 1);
      var geo = new THREE.EdgesGeometry(geometry); // or WireframeGeometry( geometry )

      // const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
      const material = new THREE.LineBasicMaterial({
        color: 0xffffff,
        linewidth: 3,
        linecap: "round", //ignored by WebGLRenderer
        linejoin: "round", //ignored by WebGLRenderer
      });
      // const cube = new THREE.Mesh(geometry, material);
      const cube = new THREE.LineSegments(geo, material);

      const material1 = new THREE.LineBasicMaterial({ color: 0x0000ff });

      const points = [];
      points.push(new THREE.Vector3(-10, 0, 0));
      points.push(new THREE.Vector3(0, 10, 0));
      points.push(new THREE.Vector3(10, 0, 0));

      const geometry1 = new THREE.BufferGeometry().setFromPoints(points);
      const line = new THREE.Line(geometry1, material1);

      scene.add(line);
      scene.add(cube);
      camera.position.z = 5;

      function animate() {
        requestAnimationFrame(animate);
        cube.rotation.x += 0.01;
        cube.rotation.y += 0.01;
        renderer.render(scene, camera);
      }

      animate();
    });

    return {
      store,
      loading,
      username,
      website,
      avatar_url,

      updateProfile,
      signOut,
    };
  },
  components: {
    Avatar,
  },
};
</script>
