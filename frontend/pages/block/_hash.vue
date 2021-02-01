<template>
  <div>
    <section>
      <b-container class="block-page main py-5">
        <div v-if="loading" class="text-center py-4">
          <Loading />
        </div>
        <template v-else-if="!parsedBlock">
          <h1 class="text-center">Block not found!</h1>
        </template>
        <template v-else>
          <div class="card mt-4 mb-3">
            <div class="card-body">
              <h4 class="text-center mb-4">
                {{ $t('details.block.block') }} #{{ formatNumber(blockNumber) }}
              </h4>
              <table class="table table-striped block-table">
                <tbody>
                  <tr>
                    <td>{{ $t('details.block.block_author') }}</td>
                    <td class="text-right">
                      <span v-if="parsedBlock.block_number === 0">
                        {{ $t('details.block.genesis') }}
                      </span>
                      <span v-else>
                        <nuxt-link
                          :to="`/account/${parsedBlock.block_author}`"
                          class="d-block"
                        >
                          <Identicon
                            :key="parsedBlock.block_author"
                            :value="parsedBlock.block_author"
                            :size="20"
                            :theme="'polkadot'"
                          />
                          <span
                            v-b-tooltip.hover
                            :title="$t('details.block.account_details')"
                          >
                            {{ shortAddress(parsedBlock.block_author) }}
                          </span>
                          <span v-if="parsedBlock.block_author_name !== ``"
                            >[ {{ parsedBlock.block_author_name }} ]</span
                          >
                        </nuxt-link>
                      </span>
                    </td>
                  </tr>
                  <tr>
                    <td>{{ $t('details.block.timestamp') }}</td>
                    <td class="text-right">
                      <p class="mb-0">
                        {{ getDateFromTimestamp(parsedBlock.timestamp) }}
                      </p>
                    </td>
                  </tr>
                  <tr>
                    <td>{{ $t('details.block.block_hash') }}</td>
                    <td class="text-right">
                      <p class="mb-0">{{ parsedBlock.block_hash }}</p>
                    </td>
                  </tr>
                  <tr>
                    <td>{{ $t('details.block.extrinsic_root') }}</td>
                    <td class="text-right">
                      <p class="mb-0">{{ parsedBlock.extrinsics_root }}</p>
                    </td>
                  </tr>
                  <tr>
                    <td>{{ $t('details.block.parent_hash') }}</td>
                    <td class="text-right">
                      <span v-if="parsedBlock.block_number === 0"> -- </span>
                      <span v-else>
                        <nuxt-link
                          :to="`/block?blockNumber=${blockNumber - 1}`"
                        >
                          {{ parsedBlock.parent_hash }}
                        </nuxt-link>
                      </span>
                    </td>
                  </tr>
                  <tr>
                    <td>{{ $t('details.block.state_root') }}</td>
                    <td class="text-right">
                      <p class="mb-0">{{ parsedBlock.state_root }}</p>
                    </td>
                  </tr>
                </tbody>
              </table>
              <b-tabs class="mt-4" content-class="mt-4" fill>
                <b-tab active>
                  <template #title>
                    <h5>{{ $t('details.block.extrinsics') }}</h5>
                  </template>
                  <template v-if="parsedExtrinsics.length > 0">
                    <table class="table table-striped">
                      <thead>
                        <tr>
                          <th>ID</th>
                          <th>{{ $t('details.block.hash') }}</th>
                          <th>{{ $t('details.block.signer') }}</th>
                          <th>{{ $t('details.block.section') }}</th>
                          <th>{{ $t('details.block.method') }}</th>
                          <th>{{ $t('details.block.args') }}</th>
                          <th>{{ $t('details.block.success') }}</th>
                        </tr>
                      </thead>
                      <tbody>
                        <tr
                          v-for="extrinsic in parsedExtrinsics"
                          :key="extrinsic.hash"
                        >
                          <td>
                            <nuxt-link
                              :to="`/extrinsic/${extrinsic.block_number}/${extrinsic.extrinsic_index}`"
                              :title="$t('details.block.account_details')"
                            >
                              {{ extrinsic.block_number }}-{{
                                extrinsic.extrinsic_index
                              }}
                            </nuxt-link>
                          </td>
                          <td>{{ shortHash(extrinsic.hash) }}</td>
                          <td>
                            <span v-if="extrinsic.signer">
                              <Identicon
                                :key="extrinsic.signer"
                                :value="extrinsic.signer"
                                :size="20"
                                :theme="'polkadot'"
                              />
                              <nuxt-link
                                :to="`/account/${extrinsic.signer}`"
                                :title="$t('details.block.account_details')"
                              >
                                {{ shortAddress(extrinsic.signer) }}
                              </nuxt-link>
                            </span>
                          </td>
                          <td>{{ extrinsic.section }}</td>
                          <td>{{ extrinsic.method }}</td>
                          <td>{{ extrinsic.args }}</td>
                          <td>
                            <i
                              v-if="extrinsic.success"
                              class="fa fa-check-circle text-success"
                              aria-hidden="true"
                            ></i>
                            <i
                              v-else
                              class="fa fa-check-circle text-danger"
                              aria-hidden="true"
                            ></i>
                          </td>
                        </tr>
                      </tbody>
                    </table>
                  </template>
                </b-tab>
                <b-tab>
                  <template #title>
                    <h5>{{ $t('details.block.system_events') }}</h5>
                  </template>
                  <template v-if="parsedEvents.length > 0">
                    <table class="table table-striped">
                      <thead>
                        <tr>
                          <th>{{ $t('details.block.section') }}</th>
                          <th>{{ $t('details.block.method') }}</th>
                          <th>{{ $t('details.block.phase') }}</th>
                          <th>{{ $t('details.block.data') }}</th>
                        </tr>
                      </thead>
                      <tbody>
                        <tr
                          v-for="event in parsedEvents"
                          :key="event.event_index"
                        >
                          <td>{{ event.section }}</td>
                          <td>{{ event.method }}</td>
                          <td>{{ event.phase }}</td>
                          <td>
                            <template
                              v-if="
                                event.section === `balances` &&
                                event.method === `Transfer`
                              "
                            >
                              <Identicon
                                :key="JSON.parse(event.data)[0]"
                                :value="JSON.parse(event.data)[0]"
                                :size="20"
                                :theme="'polkadot'"
                              />
                              <nuxt-link
                                :to="`/account/${JSON.parse(event.data)[0]}`"
                                :title="$t('details.block.account_details')"
                              >
                                {{ shortAddress(JSON.parse(event.data)[0]) }}
                              </nuxt-link>
                              <i class="fas fa-arrow-right"></i>
                              <Identicon
                                :key="JSON.parse(event.data)[1]"
                                :value="JSON.parse(event.data)[1]"
                                :size="20"
                                :theme="'polkadot'"
                              />
                              <nuxt-link
                                :to="`/account/${JSON.parse(event.data)[1]}`"
                                :title="$t('details.block.account_details')"
                              >
                                {{ shortAddress(JSON.parse(event.data)[1]) }}
                              </nuxt-link>
                              <i class="fas fa-arrow-right"></i>
                              <span class="amount">
                                {{ formatAmount(JSON.parse(event.data)[2]) }}
                              </span>
                            </template>
                            <template v-else>
                              {{ event.data }}
                            </template>
                          </td>
                        </tr>
                      </tbody>
                    </table>
                  </template>
                </b-tab>
              </b-tabs>
            </div>
          </div>
        </template>
      </b-container>
    </section>
  </div>
</template>
<script>
import gql from 'graphql-tag'
import Identicon from '@/components/Identicon.vue'
import Loading from '@/components/Loading.vue'
import commonMixin from '@/mixins/commonMixin.js'
import { network } from '@/polkastats.config.js'

export default {
  components: {
    Identicon,
    Loading,
  },
  mixins: [commonMixin],
  data() {
    return {
      loading: true,
      blockHash: this.$route.params.hash,
      blockNumber: this.$route.query.blockNumber,
      parsedBlock: undefined,
      parsedExtrinsics: [],
      parsedEvents: [],
    }
  },
  head() {
    return {
      title: this.$t('pages.block.head_title', {
        networkName: network.name,
        blockNumber: this.$route.query.blockNumber,
      }),
      meta: [
        {
          hid: 'description',
          name: 'description',
          content: this.$tc('pages.block.head_content', {
            networkName: network.name,
            blockNumber: this.$route.query.blockNumber,
          }),
        },
      ],
    }
  },
  watch: {
    $route() {
      this.blockHash = this.$route.params.hash
    },
  },
  methods: {
    getDateFromTimestamp(timestamp) {
      if (timestamp === 0) {
        return `--`
      }
      const newDate = new Date()
      newDate.setTime(timestamp * 1000)
      return newDate.toUTCString()
    },
  },
  apollo: {
    block: {
      query: gql`
        query block($block_hash: String!) {
          block(where: { block_hash: { _eq: $block_hash } }) {
            block_author
            block_author_name
            block_hash
            block_number
            extrinsics_root
            parent_hash
            state_root
            timestamp
            total_events
            total_extrincics
          }
        }
      `,
      variables() {
        return {
          block_hash: this.blockHash,
        }
      },
      result({ data }) {
        if (data.block[0]) {
          this.blockNumber = data.block[0].block_number
          this.parsedBlock = data.block[0]
          this.loading = false
        }
      },
    },
    event: {
      query: gql`
        query event($block_number: bigint!) {
          event(where: { block_number: { _eq: $block_number } }) {
            block_number
            data
            event_index
            method
            phase
            section
          }
        }
      `,
      skip() {
        return !this.blockNumber
      },
      variables() {
        return {
          block_number: this.blockNumber,
        }
      },
      result({ data }) {
        this.parsedEvents = data.event
      },
    },
    extrinsic: {
      query: gql`
        query extrinsic($block_number: bigint!) {
          extrinsic(where: { block_number: { _eq: $block_number } }) {
            block_number
            extrinsic_index
            is_signed
            signer
            section
            method
            args
            hash
            doc
            success
          }
        }
      `,
      skip() {
        return !this.blockNumber
      },
      variables() {
        return {
          block_number: this.blockNumber,
        }
      },
      result({ data }) {
        this.parsedExtrinsics = data.extrinsic
      },
    },
  },
}
</script>
<style>
.identicon {
  cursor: pointer;
}
.block-table tr td:first-child {
  width: 30%;
  font-weight: bold;
}
.block-table tr td:nth-child(2) {
  width: 70%;
}
.block-table tr td .identicon {
  display: inline-block;
}
.block-page .amount {
  color: #ef1073;
  font-weight: 700;
}
.block-page .clipboard {
  display: inline-block;
}
</style>