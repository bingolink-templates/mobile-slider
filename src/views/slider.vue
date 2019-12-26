<template>
    <div ref="wrap" class="main">
        <!-- 轮播图 -->
        <div class="slider-common" :class="[isIPhone6sp ? 'slider-mar' : '']" v-if='sliderItems.length != 0'>
            <image-slider v-bind:style="{'height': honor8 ? '192wx' : '210wx'}" auto="true" pos="1" :items="sliderItems" @onItemClick="sliderEvent" scaleType='FIT_XY' radius="10"></image-slider>
        </div>
        <div class="no-data flex-ac flex-jc" v-bind:style="{'height': honor8 ? '192wx' : '210wx'}" v-if='sliderItems.length == 0 && isShow'>
            <div class="flex-dr">
                <bui-image src="/image/sleep1.png" width="21wx" height="21wx"></bui-image>
                <text class="f26 c51 fw4 pl15 center-height ">{{isError?i18n.NoneData:i18n.ErrorLoadData}}</text>
            </div>
        </div>
    </div>
</template>

<script>
const dom = weex.requireModule("dom");
const link = weex.requireModule("LinkModule");
const linkapi = require("linkapi");
const storage = weex.requireModule('storage');
export default {
    data() {
        return {
            sliderItems: [],
            channel: new BroadcastChannel("WidgetsMessage"),
            ReplaceToDiskUri: "http://172.28.0.158:80",
            URL_PATTERN: /((http(s)?:\/\/|www\.|WWW\.)([/\w-./@?_!~$%&=:#;+\-()]*)?)/g,
            STORE_PATTERN: /(store:\/\/)/g,
            i18n: '',
            isShow: false,
            isError: true,
        };
    },
    methods: {
        _isHttpOrFile(path) {
            return (
                path.indexOf("http") == 0 ||
                path.indexOf("file") == 0 ||
                path.indexOf("data:image") == 0
            );
        },
        // 轮播图
        sliderEvent(one) {
            let url = this.sliderItems[one.pos].action;
            if (url) {
                linkapi.openLinkBroswer("", url);
            } else {
                this.$toast('地址不存在');
            }
        },
        _getContext() {
            let url = weex.config.bundleUrl;
            if (url.indexOf("?") > 0) {
                url = url.substring(0, url.indexOf("?"));
            }
            url = url.split("/").slice(0, -1).join("/");
            if (this._isHttpOrFile(url)) {
                return url;
            } else {
                if (url.indexOf("/") == 0) {
                    url = url.substring(url.indexOf("/") + 1);
                }
                url = `file:///android_asset${url == "" ? "" : "/"}${url}`;
            }
            return url;
        },
        getLastKeyStr(str, key) {
            if (!str) return "";
            return str.substring(str.lastIndexOf(key) + 1, str.length);
        },
        UrlAddParam(url, name, value) {
            var r = url;
            if (r != null && r != 'undefined' && r != "") {
                value = encodeURIComponent(value);
                var reg = new RegExp("(^|)" + name + "=([^&]*)(|$)");
                var tmp = name + "=" + value;
                if (url.match(reg) != null) {
                    r = url.replace(eval(reg), tmp);
                } else {
                    if (url.match("[?]")) {
                        r = url + "&" + tmp;
                    } else {
                        r = url + "?" + tmp;
                    }
                }
            }
            return r;
        },
        getUrl(actionParams) {
            if (!actionParams) return {};
            var sPkey = /\s/;
            var us = actionParams.split(sPkey);
            var params = {};
            for (var i = 0; i < us.length; i++) {
                var u = us[i];
                var idx = u.indexOf('=');
                if (idx > -1) {
                    params[u.substring(0, idx).trim()] = u.substring(idx + 1, u.length).trim();
                }
            }
            var url = params.url;
            delete params.url;
            for (var key in params) {
                url = this.UrlAddParam(url, key, params[key]);
            }
            return url;
        },
        getAction(action) {
            if (typeof action === 'string') {
                action = action.replace(/[\r\n]/g, ' ')
                try {
                    action = JSON.parse(action)
                } catch (e) {
                    action = eval('(' + action + ')');
                }
            }
            var pcAction = action.pc || action.android;
            if (pcAction.indexOf('[OpenUrl]') === 0) {
                return this.getUrl(pcAction);
            } else {
                return ''
            }
        },
        getStorage(callback) {
            storage.getItem('sliderJLocalData', res => {
                if (res.result == 'success') {
                    var data = JSON.parse(res.data)
                    this.isShow = true
                    this.isError = true
                    this.sliderItems = data;
                    this.broadcastWidgetHeight()
                } else {
                    callback()
                }
            })
        },
        getSliderData() {
            link.getServerConfigs([], params => {
                linkapi.get({
                    url: params.comwidgetsUri + "/carousel/list",
                    data: {
                        limit: 5
                    }
                }).then(res => {
                    this.isShow = true
                    this.isError = true
                    if (res.code == 200) {
                        try {
                            let sliderItemsArr = [];
                            for (let index = 0; index < res.data.length; index++) {
                                const element = res.data[index];
                                let sliderItemsObj = {};
                                let action = this.getAction(element.action)
                                sliderItemsObj["action"] = action
                                sliderItemsObj["title"] = element.title;
                                sliderItemsObj["url"] = element.image
                                sliderItemsObj["placeholder"] = this._getContext() + "/image/ellipsis.png";
                                sliderItemsArr.push(sliderItemsObj);
                            }
                            this.sliderItems = sliderItemsArr;
                            this.broadcastWidgetHeight();
                            storage.setItem('sliderJLocalData', JSON.stringify(sliderItemsArr))
                        } catch (error) {
                            this.error();
                        }
                    }
                }, err => {
                    this.error();
                }
                );
            }, err => {
                this.error();
            }
            );
        },
        error() {
            this.isError = false
            this.isShow = true
            this.broadcastWidgetHeight();
        },
        getComponentRect(_params) {
            dom.getComponentRect(this.$refs.wrap, (ret) => {
                this.channel.postMessage({
                    widgetHeight: ret.size.height,
                    id: _params.id
                });
            });
        },
        broadcastWidgetHeight() {
            let _params = this.$getPageParams();
            // 防止高度通知失败
            setTimeout(() => {
                this.getComponentRect(_params)
            }, 200)
            setTimeout(() => {
                this.getComponentRect(_params)
            }, 1200)
        }
    },
    created() {
        this.isIPhone6sp = WXEnvironment && (WXEnvironment.deviceModel === 'iPhone8,2' || WXEnvironment.deviceModel === 'iPhone9,2')
        this.honor8 = this.$isAndroid()
        this.$fixViewport();
        linkapi.getLanguage(res => {
            this.i18n = this.$window[res];
        });
    },
    mounted() {
        var that = this
        this.channel.onmessage = event => {
            if (event.data.action === "RefreshData") {
                this.getSliderData();
            }
        };
        this.getStorage(function () {
            that.getSliderData()
        })
    }
};
</script>

<style lang="css" src="../css/common.css"></style>
<style>
.slider-common {
    background-color: #fff;
}
.slider-mar {
    padding: 10wx;
}
.no-data {
    height: 70wx;
    flex: 1;
    background-color: #fff;
}
</style>