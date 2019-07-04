<template>
    <div ref="wrap">
        <!-- 轮播图 -->
        <div class="slider-common">
            <image-slider style="height:400px;" auto="true" pos="1" :items="sliderItems" @onItemClick="sliderEvent"
                radius="10" scaleType='FIT_XY'></image-slider>
        </div>
    </div>
</template>

<script>
    const dom = weex.requireModule("dom");
    const link = weex.requireModule("LinkModule");
    const linkapi = require("linkapi");
    export default {
        data() {
            return {
                sliderItems: [],
                channel: new BroadcastChannel("WidgetsMessage"),
                ReplaceToDiskUri: "http://172.28.0.158:80",
                URL_PATTERN: /((http(s)?:\/\/|www\.|WWW\.)([/\w-./@?_!~$%&=:#;+\-()]*)?)/g,
                STORE_PATTERN: /(store:\/\/)/g
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
                linkapi.openLinkBroswer("", url);
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
            getImageUrl(resourceUrl, accessToken, storeUri) {
                var resourceAtUrl = '';
                var storeFileId;
                var storeGetFile = storeUri + '/store/getFile?fileId=';
                if (resourceUrl.match(this.STORE_PATTERN)) {
                    if (resourceUrl.length > 20) {
                        storeFileId = resourceUrl.replace(this.STORE_PATTERN, "");
                        resourceAtUrl = storeGetFile + storeFileId + '&access_token=' + accessToken;
                    }
                } else if (resourceUrl.match(this.URL_PATTERN)) {
                    resourceAtUrl = resourceUrl;
                } else {
                    storeFileId = resourceUrl.replace(/(^.+fileId=)/g, "");
                    resourceAtUrl = storeGetFile + storeFileId + '&access_token=' + accessToken;
                }
                return resourceAtUrl;
            },
            broadcastWidgetHeight() {
                let _params = this.$getPageParams();
                setTimeout(() => {
                    dom.getComponentRect(this.$refs.wrap, ret => {
                        this.channel.postMessage({
                            widgetHeight: ret.size.height,
                            id: _params.id
                        });
                    });
                }, 200);
            },
            getToken(success, error) {
                return new Promise((resolve, reject) => {
                    link.getToken([], res => {
                        resolve(res);
                        success && success(res);
                    }, err => {
                        reject(err);
                        error && error(err);
                    }
                    );
                });
            },
            getSliderData() {
                this.getToken(token => {
                    link.getServerConfigs([], params => {
                        linkapi.get({
                            url: params.comwidgetsUri + "/carousel/list"
                        }).then(res => {
                            if (res.code == 200) {
                                let sliderItemsArr = [];
                                for (let index = 0; index < res.data.length; index++) {
                                    const element = res.data[index];
                                    let sliderItemsObj = {};
                                    let action = JSON.parse(element.action);
                                    sliderItemsObj["action"] = action.mobile_web ? action.mobile_web : action.web;
                                    sliderItemsObj["title"] = element.title;
                                    sliderItemsObj["url"] = this.getImageUrl(element.image, token.accessToken, params.storeUri);
                                    sliderItemsObj["placeholder"] = this._getContext() + "/image/logo.png";
                                    sliderItemsArr.push(sliderItemsObj);
                                }
                                this.sliderItems = sliderItemsArr;
                                this.broadcastWidgetHeight();
                            }
                        }, err => {
                            this.error();
                        }
                        );
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
                this.broadcastWidgetHeight();
            }
        },
        created() {
            linkapi.getLanguage(res => {
                this.i18n = this.$window[res];
            });
        },
        mounted() {
            this.channel.onmessage = event => {
                if (event.data.action === "RefreshData") {
                    this.getSliderData();
                }
            };
            this.getSliderData();
        }
    };
</script>

<style lang="css" src="../css/common.css"></style>
<style>
    .slider-common {
        background-color: #fff;
        height: 420px;
    }
</style>