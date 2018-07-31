<template>
  <div>
    <Input v-model="UserId" placeholder="UserId" style="width: 300px"></Input>
    <Input v-model="access_token" type="textarea" size="large" :autosize="{minRows: 1,maxRows: 2}" placeholder="access_token" style="width: 300px"></Input>
    <Select v-model="contract" placeholder="Select your contract">
      <Option v-for="contract in contracts" :key="contract" :value="contract">{{contract}}</Option>
    </Select>
    <Table border height="400" :show-header="false" :columns="headers" :data="sellOrders"></Table>
    <Table border height="400" :columns="headers" :data="buyOrders"></Table>
  </div>
</template>
<script>
export default {
  data () {
    return {
      headers: [
        {
          width: 60,
          title: 'No.',
          key: '',
          render: (h, params) => {
            return h('div', params.index + 1)
          }
        },
        {
          width: 120,
          title: 'Contract',
          key: 'Contract'
        },
        {
          width: 100,
          title: 'Side',
          key: 'Side'
        },
        {
          width: 260,
          title: 'ClientOrderId',
          key: 'ClientOrderId'
        },
        {
          width: 100,
          title: 'InsertPrice',
          key: 'InsertPrice'
        },
        {
          width: 100,
          title: 'InsertVolume',
          key: 'InsertVolume'
        },
        {
          width: 100,
          title: 'TradedVolume',
          key: 'TradedVolume'
        },
        {
          width: 120,
          title: 'OrderStatus',
          key: 'OrderStatus'
        },
        {
          width: 200,
          title: 'InsertTime',
          key: 'InsertTime'
        },
        {
          width: 80,
          title: 'Action',
          key: '',
          render: (h, params) => {
            return h('div', [
              h('Button', {
                props: {
                  type: 'primary',
                  size: 'small'
                },
                style: {
                  marginRight: '5px'
                },
                on: {
                  click: () => {
                    this.cancel(params.row)
                  }
                }
              }, 'cancel')
            ])
          }
        }
      ],
      host: 'http://119.23.43.145:7443',
      contract: 'BTC_USDT',
      contracts: ['BTC_USDT'],
      sellOrders: [],
      buyOrders: [],
      UserId: '5a52790f40a1780e52659233',
      access_token: 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiIxMzYyMTY4NzkwOCIsImV4cCI6MTUzMTc1MjcyMDg1NX0.UkPCVF93M7uUKs9_7sIYKxKcpj3CIPZFPRy6HhZxnsc'

    }
  },
  created () {
    this.getContract()
    this.getOrders()
    setInterval(() => {
      this.getOrders()
    }, 10000)
  },
  watch: {
    contract (newval, oldval) {
      console.log(newval, oldval)
      this.getOrders()
    }
  },
  methods: {
    getContract () {
      this.$ajax.get(`${this.host}/contract/precision`).then(rs => {
        console.log(rs.data)
        const data = rs.data
        if (data.status) {
          this.contracts = data.data.map(item => item.TransactionCode)
          console.log(this.contracts)
        }
      })
    },
    getOrders () {
      this.$ajax.get(`${this.host}/rsporder/list`, {
        timeout: 5000,
        params: {
          Contract: this.contract,
          UserId: this.UserId,
          access_token: this.access_token
        }
      }).then(rs => {
        console.log(rs)
        const data = rs.data
        if (data.status) {
          const orders = data.data.filter(order => order.OrderStatus === 'Received' || order.OrderStatus === 'PartTradingQueueing')
          // console.log(orders)
          this.sellOrders = orders.filter(order => order.Side === 'Sell').sort((pre, cur) => new Date(cur.InsertPrice) - new Date(pre.InsertPrice))
          this.buyOrders = orders.filter(order => order.Side === 'Buy').sort((pre, cur) => new Date(cur.InsertPrice) - new Date(pre.InsertPrice))
        } else {
          this.$Notice.error({
            title: data.errCode,
            desc: data.message
          })
        }
      }).catch(e => {
        console.log(e)
        this.$Notice.error({
          title: e.title || e.message,
          desc: e.title || e.message || e.desc
        })
      })
    },

    cancel (order) {
      console.log(order)
      this.$ajax.post(`${this.host}/order/cancel`, {
        ClientOrderId: order.ClientOrderId,
        UserId: this.UserId,
        access_token: this.access_token
      }).then(rs => {
        const data = rs.data
        if (data.status) {
          console.log('reload orders')
          this.getOrders()
        }
        console.log(data)
      }).catch(e => {
        console.log(e)
      })
    }

  }
}
</script>
