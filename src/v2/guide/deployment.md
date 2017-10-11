---
title: Triển khai cho môi trường production
type: guide
order: 401
---

## Bật chế độ production

Trong quá trình phát triển, Vue cung cấp rất nhiều cảnh báo nhằm giúp bạn tránh những lỗi và nguy cơ tiềm ẩn thường gặp. Tuy nhiên, những dòng cảnh báo này lại trở nên vô ích trong môi trường production và làm phình to kích thước ứng dụng. Thêm nữa, một vài trong số những phần kiểm tra cảnh báo có runtime costs nhỏ có thể được bỏ qua trong chế độ production.

### Không sử dụng build tools (công cụ build)

Nếu bạn đang sử dụng bản đầy đủ, nghĩa là chèn trực tiếp Vue bằng thẻ script và không sử dụng build tool, hãy đảm bảo đó là bản đã minified (thu nhỏ) cho môi trường production. Cả hai phiên bản đều có thể được tìm thấy tại [Hướng dẫn cài đặt](installation.html#Direct-lt-script-gt-Include).

### Sử dụng build tools

Nếu bạn đang sử dụng build tool như Webpack hoặc Browserify, chế độ production sẽ được xác định bởi biến môi trường `process.env.NODE_ENV` bên trong mã nguồn của Vue, lưu ý giá trị mặc định của biến này là chế độ development (đang phát triển). Cả hai build tool đều cung cấp các cách thức để ghi đè giá trị của biến này nhằm bật chế độ production, khi đó minifiers sẽ bỏ đi các cảnh báo trong quá trình build. Tất cả templates (bản mẫu) của `vue-cli` đều đã được pre-configured (cấu-hình-trước) cho mục đích này, dù vậy vẫn sẽ có ích nếu bạn hiểu cách thức hoạt động, như mô tả dưới đây:

#### Webpack

Sử dụng [DefinePlugin](https://webpack.js.org/plugins/define-plugin/) của Webpack để xác định môi trường production, những dòng cảnh báo sẽ được tự động bỏ đi trong quá trình minification (thu nhỏ) được tiến hành bởi UglifyJS. Dưới đây là một ví dụ:

``` js
var webpack = require('webpack')

module.exports = {
  // ...
  plugins: [
    // ...
    new webpack.DefinePlugin({
      'process.env': {
        NODE_ENV: '"production"'
      }
    })
  ]
}
```

#### Browserify

- Chạy lệnh bundling (đóng gói) với giá trị biến môi trường `NODE_ENV` là `"production"`. Cách này sẽ giúp cho `vueify` bỏ qua hot-reload và phần mã trợ giúp cho quá trình phát triển.

- Áp dụng một global [envify](https://github.com/hughsk/envify) transform cho bundle. Cách này sẽ giúp minifier (công cụ thu nhỏ) lược đi toàn bộ những cảnh báo trong mã nguồn của Vue được gói trong các coditional block (khối điều kiện) dùng để kiểm tra biến môi trường. Ví dụ:

  ``` bash
  NODE_ENV=production browserify -g envify -e main.js | uglifyjs -c -m > build.js
  ```

#### Rollup

Dùng [rollup-plugin-replace](https://github.com/rollup/rollup-plugin-replace):

``` js
const replace = require('rollup-plugin-replace')

rollup({
  // ...
  plugins: [
    replace({
      'process.env.NODE_ENV': JSON.stringify( 'production' )
    })
  ]
}).then(...)
```

## Pre-Compiling templates (Mẫu biên-dịch-trước)

Khi sử dụng in-DOM templates hoặc in-JavaScript template strings, việc biên dịch template-to-render-function diễn ra ngay trong quá trình chạy ứng dụng. Tốc độ thường là đủ nhanh cho hầu hết các trường hợp, tuy nhiên tốt nhất vẫn nên tránh điều này nếu ứng dụng của bạn đặt nặng về performance (hiệu suất).

Cách dễ nhất để pre-compile templates là sử dụng [Single-File Components](single-file-components.html) - những cài đặt build liên quan sẽ tự động thực hiện pre-compilation (biên-dịch-trước), mã sau khi build sẽ bao gồm những hàm render đã được dịch thay vì raw template string (chuỗi mẫu thô).

Nếu bạn dùng Webpack và muốn tách biệt giữa JavaScript and template files, có thể sử dụng [vue-template-loader](https://github.com/ktsn/vue-template-loader), công cụ này cũng sẽ chuyển hoá files template thành hàm render của Javascript trong quá trình build.

## Trích xuất CSS của component

Khi sử dụng Single-File Components, CSS bên trong components được chèn động vào các thẻ `<style>` thông qua JavaScript. Runtime costs cho cách này là nhỏ, và nếu bạn sử dụng server-side rendering (render từ-phía-server) sẽ gây ra hiện tượng "flash of unstyled content" ("hiển thị nội dung thô trong giây lát"). Trích xuất CSS ở tất cả các components và gom vào một file sẽ tránh được những vấn đề nêu trên, ngoài ra việc caching và thu nhỏ CSS cũng sẽ tốt hơn.

Tham khảo tài liệu của từng build tool dưới đây để hiểu cách thức hoạt động:

- [Webpack + vue-loader](https://vue-loader.vuejs.org/en/configurations/extract-css.html) (template webpack của `vue-cli` đã cấu hình sẵn cho cách này)
- [Browserify + vueify](https://github.com/vuejs/vueify#css-extraction)
- [Rollup + rollup-plugin-vue](https://vuejs.github.io/rollup-plugin-vue/#/en/2.3/?id=custom-handler)

## Theo dõi lỗi runtime

Một lỗi runtime xảy ra trong quá trình render component sẽ được truyền đến hàm global config (cấu hình toàn cục) `Vue.config.errorHandler`, với điều kiện hàm này đã được thiết lập từ trước. Một ý tưởng hợp lí khác là kết hợp hook này với một dịch vụ error-tracking (theo-dõi-lỗi) như Sentry, dịch vụ này còn cung cấp sẵn một giải pháp tích hợp chính thức dành cho Vue.
