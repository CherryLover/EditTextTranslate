# EditTextTranslate

## BBD

我受够了自己不会英语，然后各种尴尬的场景了，尤其是现在各种 AI 的 Prompt 的英文输出能力更强。所以刚才突发奇想，能不能通过快捷键，将输入的内容翻译，再回填到任意页面的输入法，结果是不可以……


然后我找到了曲线救国的方案：输入完成后，选中输入内容，触发快捷键，调用翻译服务，获取结果，写入到系统剪贴板，发送通知，粘贴文本。


多说一点点🤏 翻译服务比较麻烦，本来是想，调用微软的服务或者用 chatgpt 来翻译，但是吧，前者我现在沉浸式翻译也在用，而且免费额度低，后者吧，花钱，肉疼。


然后吧，突然想到 cloudflare 的 worker 现在支持 AI 了，那就试试？然后用的是 cloudflare 的 AI 做的翻译。

整体的代码量很少很少，比我写过的所有其他项目都少，但是带给我的体验确实不同凡响的。

这个仓库比较简单，就是放一个 Alfred 的 workflow，再加上 Cloudflare Worker 的代码，仅此而已。

PS：如果想用，需要 Mac 电脑，命令行装有 curl（Mac 上好像默认就有），有 Alfred 软件，能访问 Cloudflare Worker。

PPS：我把地址放出来，如果你不用 Alfred，自己写一个其他的，一个网络请求解决。

PPPS：不仅支持中英文，还有其他语言，具体参见：https://developers.cloudflare.com/workers-ai/models/translation/

## 共用

> 更新：2024-01-02
>
> 我又顺手做了个快捷指令，这样一来，你输入中文之后，复制，然后点一下桌面上的快捷指令就能自动翻译，然后回到输入框，粘贴即可！！！🥳🥳🥳
>
> 苹果快捷指令在👇
> 
> https://www.icloud.com/shortcuts/cfd905a29f0845e9905c3150318817cc

Alfred Workflow 下载地址：https://github.com/CherryLover/EditTextTranslate/blob/main/EditTextTranslate.alfredworkflow

接口访问地址：https://translate.odjwzonline.uk 用 POST 提交

用的人给个 star 🌟 哈

```JSON
{
  "text": "你是谁",
  "source_lan": "chinese",
  "target_lan": "english"
}
```

## 自建
创建 worker 时记得选这个。
![image](https://github.com/CherryLover/EditTextTranslate/assets/18376501/2f7783ee-7696-4ba9-a59a-8284175cc893)

然后复制粘贴以下代码
Cloudflare Worker 代码
```JavaScript
import { Ai } from './vendor/@cloudflare/ai.js';

export default {
	async fetch(request, env) {
		if (request.method !== 'POST') {
			return new Response('Method Not Allowed.', { status: 405 });
		}
		let json = await request.json();
    console.log(JSON.stringify(json))
		const ai = new Ai(env.AI);

		const inputs = {
			text: json['text'],
			source_lang: json['source_lan'],
			target_lang: json['target_lan']
		};
		const response = await ai.run('@cf/meta/m2m100-1.2b', inputs);

		return new Response(response.translated_text, {	headers: { 'content-type': 'text/plain' } });
	}
};
```
