<template>
  <div>
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
      sellOrders: [],
      buyOrders: [],
      UserId: '5a52790f40a1780e52659233',
      access_token: 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiIxMzYyMTY4NzkwOCIsImV4cCI6MTUzMTc1MjcyMDg1NX0.UkPCVF93M7uUKs9_7sIYKxKcpj3CIPZFPRy6HhZxnsc'

    }
  },
  created () {
    this.getOrders()
    setInterval(() => {
      this.getOrders()
    }, 10000)
  },
  methods: {
    getOrders () {
      this.$ajax.get('http://119.23.43.145:7443/rsporder/list', {
        timeout: 5000,
        params: {
          Contract: 'ETC_USDT',
          UserId: this.UserId,
          access_token: this.access_token
        }
      }).then(rs => {
        const data = rs.data
        if (data.status) {
          const orders = data.data.filter(order => order.OrderStatus === 'Received' || order.OrderStatus === 'PartTradingQueueing')
          // console.log(orders)
          this.sellOrders = orders.filter(order => order.Side === 'Sell').sort((pre, cur) => new Date(cur.InsertPrice) - new Date(pre.InsertPrice))
          this.buyOrders = orders.filter(order => order.Side === 'Buy').sort((pre, cur) => new Date(cur.InsertPrice) - new Date(pre.InsertPrice))
        }
      }).catch(e => {
        console.log(e)
      })
    },

    cancel (order) {
      console.log(order)
      this.$ajax.post('http://119.23.43.145:7443/order/cancel', {
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
