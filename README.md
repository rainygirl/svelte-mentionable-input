# CJKMentionableInput.svelte

ContentEditable input component that automatically colors mentions and hashtags.

- works smoothly despite CJK (Chinese, Japanese, and Korean) letters and various IME bugs that exist in Composing mode.
- tested with Chrome and Safari on MacOS.

멘션, 해시태그를 자동으로 색칠해주는 ContentEditable 입력 컴포넌트입니다. 

- Composing 모드가 존재하는 CJK (중국어, 일본어, 한국어) 글자와 다양한 IME 버그에도 불구하고 원활히 작동하는 입력 컴포넌트입니다.
- MacOS, Chrome 과 Safari 브라우저에서 테스트되었습니다.

## Screenshot

![CJKMentionableInput.svelte Screenshot](https://user-images.githubusercontent.com/1021138/216605658-a3b95a51-964d-40a1-ad49-23608c8e4a6f.gif)

## Getting Started

```
//page.svelte

<script>
	import CJKMentionableInput from './CJKMentionableInput.svelte';
	let textValue;
</script>

<section>
	<CJKMentionableInput bind:value={textValue} />
</section>
```

## Contributing
Feel free to fork & contribute!

## License
CJKMentionableInput.svelte is released under the MIT license.

## Credits

* [Lee JunHaeng aka rainygirl](https://rainygirl.com).

