<script>
  export default {
    render() { },

    props: {
      renderItem: {
        type: Function,
        required: true,
        validator(value) {
          const component = value(0);
          if (component._isVue && !component._isMounted) {
            component.$destroy();
            return true;
          }
          return false;
        }
      },
      length: {
        type: Number,
        required: true
      },
      calculateItemHeight: {
        type: Function,
        default: undefined
      }
    },

    data() {
      return {
        provider: null
      };
    },

    methods: {
      _setup() {
        const delegate = new this.$ons._ons._internal.LazyRepeatDelegate({
          calculateItemHeight: this.calculateItemHeight,
          createItemContent: i => this.renderItem(i).$mount().$el,
          destroyItem: (i, { element }) => element.__vue__.$destroy(),
          countItems: () => this.length
        }, null);

        this.provider = new this.$ons._ons._internal.LazyRepeatProvider(this.$parent.$el, delegate);
      },
      refresh() {
        return this.provider.refresh();
      }
    },

    watch: {
      renderItem() {
        this._setup();
      },
      length() {
        this._setup();
      },
      calculateItemHeight() {
        this._setup();
      }
    },

    mounted() {
      this._setup();
      this.$vnode.context.$on('refresh', this.refresh);
    },

    beforeDestroy() {
      this.$vnode.context.$off('refresh', this.refresh);
      this.provider && this.provider.destroy();
    }
  };
</script>
