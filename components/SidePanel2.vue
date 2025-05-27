<template>

  <div class="content">
      <div class="sidebarcontainer">
      <div class="sidebarsection">
          <div class="sidebar">
          <!-- Menu Section -->
          <div class="menu">
            <!-- <div class="searchbar" @click="">   
          <input type="text" v-model="query" placeholder="Search" @keyup.enter="searchMovies" spellcheck="false">
          <div @click="searchMovies" :disabled="loading" class="searchbutton">
            <svg xmlns='http://www.w3.org/2000/svg'  viewBox='0 0 24 24' fill='#000000' width='16' height='16'><path d="M10 18a7.952 7.952 0 0 0 4.897-1.688l4.396 4.396 1.414-1.414-4.396-4.396A7.952 7.952 0 0 0 18 10c0-4.411-3.589-8-8-8s-8 3.589-8 8 3.589 8 8 8zm0-14c3.309 0 6 2.691 6 6s-2.691 6-6 6-6-2.691-6-6 2.691-6 6-6z"></path></svg>
          </div>
        </div> -->
            <NuxtLink v-for="item in menuItems" :key="item.path" :to="item.path" active-class="active">
              <div class="home" @click="handleItemClick">
                <div class="menu-item">
      <span class="icon" v-html="item.icon"></span>
      <p :class="{ 'is-expanded': isMenuOpen }">{{ item.label }}</p>
    </div>              </div>
            </NuxtLink>
          </div>
                 
          </div>
      </div>
      </div>
  </div>
</template>

<script setup>
import { useToggle } from '~/composables/useToggle';

const { isMenuOpen, closeMenu } = useToggle();

const menuItems = [
  { label: 'Dashboard', path: '/', icon: `<svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 256 256"><path d="M216,36H40A20,20,0,0,0,20,56V200a20,20,0,0,0,20,20H216a20,20,0,0,0,20-20V56A20,20,0,0,0,216,36Zm-4,24V92H44V60ZM44,116H92v80H44Zm72,80V116h96v80Z"></path></svg>` },
  { label: 'Vehicle Compliance', path: '/',  icon: `<svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 256 256"><path d="M208,36H48A20,20,0,0,0,28,56v56c0,54.29,26.32,87.22,48.4,105.29,23.71,19.39,47.44,26,48.44,26.29a12.1,12.1,0,0,0,6.32,0c1-.28,24.73-6.9,48.44-26.29,22.08-18.07,48.4-51,48.4-105.29V56A20,20,0,0,0,208,36Zm-4,76c0,35.71-13.09,64.69-38.91,86.15A126.28,126.28,0,0,1,128,219.38a126.14,126.14,0,0,1-37.09-21.23C65.09,176.69,52,147.71,52,112V60H204ZM79.51,144.49a12,12,0,1,1,17-17L112,143l47.51-47.52a12,12,0,0,1,17,17l-56,56a12,12,0,0,1-17,0Z"></path></svg>` },
  { label: 'Vehicle Maintenance', path: '/VehicleMaintenance', icon: `<svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 256 256"><path d="M236,100H225L190.83,65.86A19.86,19.86,0,0,0,176.69,60H144V44h20a12,12,0,0,0,0-24H100a12,12,0,0,0,0,24h20V60H64A20,20,0,0,0,44,80v48H28V108a12,12,0,0,0-24,0v64a12,12,0,0,0,24,0V152H44v16.69a19.86,19.86,0,0,0,5.86,14.14l39.31,39.31A19.86,19.86,0,0,0,103.31,228h73.38a19.86,19.86,0,0,0,14.14-5.86L225,188h11a20,20,0,0,0,20-20V120A20,20,0,0,0,236,100Zm-4,64H220a12,12,0,0,0-8.49,3.51L175,204H105L68,167V84H175l36.48,36.49A12,12,0,0,0,220,124h12Z"></path></svg>` },
  { label: 'Job Records & Overview', path: '/', icon: '<svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" fill="#000000" viewBox="0 0 256 256"><path d="M172,124a12,12,0,0,1-12,12H96a12,12,0,0,1,0-24h64A12,12,0,0,1,172,124Zm-12,28H96a12,12,0,0,0,0,24h64a12,12,0,0,0,0-24ZM220,40V200a36,36,0,0,1-36,36H72a36,36,0,0,1-36-36V40A12,12,0,0,1,48,28H72V24a12,12,0,0,1,24,0v4h20V24a12,12,0,0,1,24,0v4h20V24a12,12,0,0,1,24,0v4h24A12,12,0,0,1,220,40ZM196,52H184v4a12,12,0,0,1-24,0V52H140v4a12,12,0,0,1-24,0V52H96v4a12,12,0,0,1-24,0V52H60V200a12,12,0,0,0,12,12H184a12,12,0,0,0,12-12Z"></path></svg>' },
  // { label: 'Client Profitability', path: '/' },
  // { label: 'Driver and Staff Records', path: '/' },
  { label: 'Settings', path: '/',     icon: `<svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 256 256"><path d="M128,76a52,52,0,1,0,52,52A52.06,52.06,0,0,0,128,76Zm0,80a28,28,0,1,1,28-28A28,28,0,0,1,128,156Zm92-27.21v-1.58l14-17.51a12,12,0,0,0,2.23-10.59A111.75,111.75,0,0,0,225,71.89,12,12,0,0,0,215.89,66L193.61,63.5l-1.11-1.11L190,40.1A12,12,0,0,0,184.11,31a111.67,111.67,0,0,0-27.23-11.27A12,12,0,0,0,146.3,22L128.79,36h-1.58L109.7,22a12,12,0,0,0-10.59-2.23A111.75,111.75,0,0,0,71.89,31.05,12,12,0,0,0,66,40.11L63.5,62.39,62.39,63.5,40.1,66A12,12,0,0,0,31,71.89,111.67,111.67,0,0,0,19.77,99.12,12,12,0,0,0,22,109.7l14,17.51v1.58L22,146.3a12,12,0,0,0-2.23,10.59,111.75,111.75,0,0,0,11.29,27.22A12,12,0,0,0,40.11,190l22.28,2.48,1.11,1.11L66,215.9A12,12,0,0,0,71.89,225a111.67,111.67,0,0,0,27.23,11.27A12,12,0,0,0,109.7,234l17.51-14h1.58l17.51,14a12,12,0,0,0,10.59,2.23A111.75,111.75,0,0,0,184.11,225a12,12,0,0,0,5.91-9.06l2.48-22.28,1.11-1.11L215.9,190a12,12,0,0,0,9.06-5.91,111.67,111.67,0,0,0,11.27-27.23A12,12,0,0,0,234,146.3Zm-24.12-4.89a70.1,70.1,0,0,1,0,8.2,12,12,0,0,0,2.61,8.22l12.84,16.05A86.47,86.47,0,0,1,207,166.86l-20.43,2.27a12,12,0,0,0-7.65,4,69,69,0,0,1-5.8,5.8,12,12,0,0,0-4,7.65L166.86,207a86.47,86.47,0,0,1-10.49,4.35l-16.05-12.85a12,12,0,0,0-7.5-2.62c-.24,0-.48,0-.72,0a70.1,70.1,0,0,1-8.2,0,12.06,12.06,0,0,0-8.22,2.6L99.63,211.33A86.47,86.47,0,0,1,89.14,207l-2.27-20.43a12,12,0,0,0-4-7.65,69,69,0,0,1-5.8-5.8,12,12,0,0,0-7.65-4L49,166.86a86.47,86.47,0,0,1-4.35-10.49l12.84-16.05a12,12,0,0,0,2.61-8.22,70.1,70.1,0,0,1,0-8.2,12,12,0,0,0-2.61-8.22L44.67,99.63A86.47,86.47,0,0,1,49,89.14l20.43-2.27a12,12,0,0,0,7.65-4,69,69,0,0,1,5.8-5.8,12,12,0,0,0,4-7.65L89.14,49a86.47,86.47,0,0,1,10.49-4.35l16.05,12.85a12.06,12.06,0,0,0,8.22,2.6,70.1,70.1,0,0,1,8.2,0,12,12,0,0,0,8.22-2.6l16.05-12.85A86.47,86.47,0,0,1,166.86,49l2.27,20.43a12,12,0,0,0,4,7.65,69,69,0,0,1,5.8,5.8,12,12,0,0,0,7.65,4L207,89.14a86.47,86.47,0,0,1,4.35,10.49l-12.84,16.05A12,12,0,0,0,195.88,123.9Z"></path></svg>`
  
},

];

// Close sidebar when a menu item is clicked
const handleItemClick = () => {
  closeMenu();
};
</script>


<style lang="scss" scoped>
    @use "@/assets/sass/variables" as *; // Import variables



.content {
  // border: solid green ;
  height: 100vh;
  overflow: hidden;
  overflow-y: scroll;
  
  .sidebarcontainer {
    // overflow: scroll;
      // border: solid yellow;
      background-color: $bg-white; /* Solid black background */
      width: 21rem;
      height: 130vh;
      // padding-inline: 1rem;
      outline: none; /* Remove the default outline */
      box-shadow: 0 0 0 0.1px ;   //remove outline and use box shadow for thinnest line 
      


  .sidebarsection {
      // height: 50rem;
      width: 100%;
      // border: solid blue;
      
      .sidebar {
        display: flex;
        flex-direction: column;
        gap: 2rem;
          // padding-top: 1.5rem;
          // border: solid blue;
          .logo {
            padding-left:1rem ;
            // letter-spacing: 6px;
                font-size: 12px;
                position: relative; /* Ensure proper layering */
                background: transparent;
                // border: solid;
                height: 3.2rem;
                display: flex;
                align-items: center;

                h1 {
                    color: $primarycolorblack;
                    position: relative; /* Ensure it stays in front */
          /* Make it behave like a button */
                }
          }
          .menu {
              // border: solid;
              display: flex;
              flex-direction: column;
              // border-bottom: solid 1px rgba(255, 255, 255, 0.162);
              padding-bottom: 0.5rem;
              padding-top: 2rem;
              // gap: 1rem;
              .searchbar {

                  // min-width: 22rem;
                  width: 100%;
                  display: flex;
                  align-items: center;
                  position: relative;
                  border: none;
                  outline: none;
                  padding-inline: 1rem;
                  // border: solid black;


                  input {
                      padding-inline: 1rem;
                      width: 100%;
                      border-radius: 6px;
                      border: none ;
                      // outline: none;
                      height: 2.5rem;
                      background-color: $bg-offwhite;
                    color: rgb(225, 213, 213);
                    font-size: 16px;

                    &:focus {
                      outline: 1px solid $bg-secondary; /* Change to your desired color */
                      }


                    }
                    .searchbutton {
                      position: absolute;
                      right: 1.5rem;
                      // top: 0.5rem;
                      // padding-inline: 0.5rem;
                      cursor: pointer;
                      // border: solid;
                      height: fit-content;
                      border: none;
                      
                      svg {
                        
                        fill: $text-accent;

                        &:hover {
                        fill: rgb(225, 213, 213);


                        }
                      }

                      

                      
                    }

                  }
                                .home {
                  // border: solid ;
                  padding-top: 0.5rem;
                  // border-radius: 0.5rem;
                  padding-bottom: 0.5rem;
                  display: flex;
                  gap: 0.5rem;
                  align-items: center;
                  cursor: pointer;
                  height: 3.5rem;
                  border-bottom: solid 1px $line-grey; ;
                  
                  
                  p {
                    -webkit-tap-highlight-color: transparent;
                    
                    // padding-left: 1.5rem;
                      font-size: 16px;
                      font-weight: 500;
                      color: $text-dark;
                    }
                    p:active {
                      
                      color: $primarycolorblack;
                  }

              }

              // .active {
              //     border-radius: 0.5rem;
              //     background-color: $primarycolorblack;

              //     p {
              //         color: $bgcolordeepwhite;
              //     }

              // }
              
          }
          }
          }
          }

        


}


@media (max-width: 800px) {
  .content {

      .sidebarcontainer {
          // width: fit-content;
          // height: 100vh;

          .sidebar {
            
            .menu {
              padding-left: 1rem;

            }
          }

          .menu-item {
            display: flex;
            align-items: center;
            gap: 1rem;

            .icon {
              width: 30px;
              height: 30px;
              display: flex;
              fill: $text-dark;

            }
          }
      }
  }

} 




</style>


