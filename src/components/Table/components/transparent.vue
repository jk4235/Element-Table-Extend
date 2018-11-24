<script>
export default {
  name: 'Transparent',
  functional: true,
  props: {
    vNode: {
      type: Object,
      default: () => {}
    }
  },
  render (h, context) {
    const props = context.props
    return props.vNode
  }
}
</script>

<style scoped>

</style>
