<template>
  <div class="follow-wrapper">
    <div class="columns is-mobile has-text-centered">
      <div class="column level-item">
        <div>
          <p class="title">{{ followingCount }}</p>
          <p class="heading">Fans</p>
        </div>
      </div>
      <div v-if="service === 'users'" class="column level-item" style="opacity: .25">
        <div>
          <p class="title">{{ 0 }}</p>
          <p class="heading">Freunde</p>
        </div>
      </div>
    </div>
    <div v-if="showButtons" class="columns is-mobile field has-text-centered">
      <div class="column control has-text-centered">
        <hc-button color="button is-fullwidth"
                   :class="{'is-primary': !follow.isLoading && !follow.isPending && !follow.isFollowing}"
                   @click="toggleFollow"
                   :disabled="follow.isPending"
                   :isLoading="follow.isPending">
          <template v-if="follow.isFollowing">
            <hc-icon icon="check" class="icon-left" /> {{ $t('component.follow.buttonLabelUnFollow') }}
          </template>
          <template v-else>
            <hc-icon icon="bell" class="icon-left" /> {{ $t('component.follow.buttonLabelFollow') }}
          </template>
        </hc-button>
      </div>
      <div v-if="service === 'users'" class="column control has-text-centered" style="opacity: .25; pointer-events: none;">
        <hc-button color="button" @click="toggleConnected" :disabled="true">
          <template v-if="connected">
            <hc-icon icon="user-times" class="icon-left" /> Trennen
          </template>
          <template v-else>
            <hc-icon icon="user" class="icon-left" /> Verbinden
          </template>
        </hc-button>
      </div>
    </div>
  </div>
</template>

<script>
  import { mapGetters } from 'vuex'

  export default {
    name: 'hc-follow-buttons',
    props: {
      showButtons: {
        type: Boolean,
        default: false
      },
      entity: {
        type: Object,
        required: true
      },
      service: {
        type: String, // users, organizations
        default: 'users'
      }
    },
    data () {
      return {
        connected: false,
        followingCount: 0
      }
    },
    mounted () {
      this.followingCount = this.entity.followersCounts ? (this.entity.followersCounts.users || 0) : 0
      this.$store.dispatch('connections/syncFollow', {
        userId: this.loggedInUser._id,
        foreignId: this.entity._id,
        foreignService: this.service
      })
    },
    computed: {
      ...mapGetters({
        follow: 'connections/follow',
        loggedInUser: 'auth/user'
      })
    },
    methods: {
      async toggleFollow () {
        if (this.follow._id) {
          await this.$store.dispatch('connections/unfollow', {
            _id: this.follow._id
          })
          this.followingCount--
          this.$snackbar.open({
            message: this.$t('component.follow.messageUnFollowSuccess', {
              name: this.entity.name || this.$t('component.contribution.creatorUnknown')
            })
          })
        } else {
          await this.$store.dispatch('connections/follow', {
            foreignId: this.entity._id,
            foreignService: this.service
          })
          this.followingCount++
          this.$snackbar.open({
            message: this.$t('component.follow.messageFollowSuccess', {
              name: this.entity.name || this.$t('component.contribution.creatorUnknown')
            })
          })
        }
      },
      toggleConnected () {
        this.connected = !this.connected
        this.$snackbar.open({
          message: 'Connected!'
        })
      }
    }
  }
</script>

<style lang="scss" scoped>
  @import "assets/styles/utilities";


  .level-item {
    user-select: none;
  }

  .follow-wrapper {
    margin-top: 1rem;
    margin-bottom: -0.5rem;
    padding: 0.8em 0.5em 0.6em;
    border-style: solid;
    border-color: lighten($grey-lighter, 5%);
    border-width: 1px 0;
    border-bottom: none;

    .title {
      font-size: 1.7em;
      // color: $primary;
    }

    strong {
      font-size: 1.3em;
      padding-bottom: 1.5em;
      // color: $primary;
    }

    .columns.field {
      margin-top: -2rem;
    }

    button.button {
      width: 100% !important;
    }
  }
</style>
