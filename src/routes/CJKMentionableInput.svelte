<script lang="ts">
	import { onMount } from 'svelte';

	export let value = '<p><br></p>';
	let keyUpDebt: Function | null = null;

	const createEl = (tag: string) => {
			return document.createElement(tag);
		},
		Caret = {
			getSelRange: () => {
				const sel = window.getSelection(),
					range = sel?.getRangeAt(0);
				return { sel, range };
			},
			getCurrentEl: (): HTMLElement | null | undefined => {
				let { range } = Caret.getSelRange();
				const ca = range?.commonAncestorContainer;
				if (ca?.nodeType === 1) return ca as HTMLElement;
				return ca?.parentElement;
			},
			insertBefore: (dom: HTMLElement) => {
				const sel = window.getSelection();
				if (!sel?.getRangeAt || !sel.rangeCount) return;
				const range = sel.getRangeAt(0);
				range.deleteContents();
				range.insertNode(dom);
			},
			insertAfter: (referenceNode: HTMLElement, newNode: HTMLElement | Node): void => {
				const pn = referenceNode.parentNode;
				if (referenceNode.nextSibling) {
					pn?.insertBefore(newNode, referenceNode.nextSibling);
				} else {
					pn?.appendChild(newNode);
				}
			},
			moveTo: (target: HTMLElement, offset: number) => {
				const sel = window.getSelection();
				const range = new Range();
				range?.setStart(target, offset);
				range?.setEnd(target, offset);
				range?.collapse(false);
				sel?.removeAllRanges();
				sel?.addRange(range);
			},
			moveEndIn: (dom: HTMLElement) => {
				if (typeof document.createRange == 'undefined') return;
				const range = document.createRange();
				range.selectNodeContents(dom);
				range.collapse(false);
				const sel = window.getSelection();
				sel?.removeAllRanges();
				sel?.addRange(range);
			},
			moveOut: (e, blank?: HTMLElement | Node | undefined) => {
				const currentEl = Caret.getCurrentEl();
				if (!currentEl) return;
				if (!blank) blank = document.createTextNode(' \u200b');

				Caret.insertAfter(currentEl, blank);
				const target = currentEl.nextSibling;
				if (target === null) return;

				const escape = (e) => {
					if (e.type === 'keyup') {
						currentEl.innerHTML = currentEl.innerHTML.replace(/&nbsp;/g, '').trim();
						(specialKeys + '#@').split('').forEach((d) => {
							currentEl.innerHTML =
								currentEl.innerHTML.substring(0, 1) +
								currentEl.innerHTML.substring(1).replaceAll(d, '');
						});
					}
					const sel = window.getSelection();
					const range = new Range();
					if (blank?.nodeType === 3) {
						range?.setStart(target, 0);
						range?.setEnd(target, target.nodeValue?.length || 0);
					} else {
						range?.selectNode(target);
					}
					range?.collapse(false);
					sel?.removeAllRanges();
					sel?.addRange(range);
				};
				if (e.isComposing) keyUpDebt = escape;
				else escape(e);

				keyUpDebt = escape;
			},
			isInSpan: (): boolean => {
				return Caret.getCurrentEl()?.tagName === 'SPAN';
			}
		},
		specialKeys = '~!@$%^&*()_+-=`"\'/,.<>[]{};:?|\\',
		keyDown = (e: KeyboardEvent) => {
			if (Caret.isInSpan()) {
				const cel = Caret.getCurrentEl(),
					currentMode = cel?.textContent?.substring(0, 1);

				if ((e.key === '#' && currentMode === '#') || (e.key === '@' && currentMode === '@')) {
					e.preventDefault();
					return false;
				}
				if (specialKeys.split('').includes(e.key)) {
					e.preventDefault();
					if (cel?.innerText.length === 1) return false;
					Caret.moveOut(e);
					return false;
				}
				if (e.key === ' ' || e.key === 'Enter') {
					e.preventDefault();
					if (cel?.innerHTML === '#') return false;
					else if (cel?.innerHTML === '#.&nbsp;') {
						cel.innerHTML = '#';
						Caret.moveEndIn(cel);
						return;
					}
					if (e.key === 'Enter' && !e.isComposing) Caret.moveOut(e, createEl('br'));
					else Caret.moveOut(e);

					return false;
				} else return true;
			}
			const { range } = Caret.getSelRange();
			if (range?.collapsed) {
				const container = range?.commonAncestorContainer,
					beforeChar = container.textContent && container.textContent[range.startOffset - 1];

				if (
					['#', '@'].includes(e.key) &&
					!e.isComposing &&
					((container as HTMLElement).tagName === 'P' ||
						typeof beforeChar === 'undefined' ||
						(beforeChar &&
							(['#', '@'].includes(beforeChar) ||
								[32, 160, 8203].includes(beforeChar.charCodeAt(0)))))
				) {
					e.preventDefault();
					const span = createEl('span');
					span.innerHTML = e.key;
					Caret.insertBefore(span);
					const ps = span.previousSibling;
					if (e.key === beforeChar && ps) {
						/*
						If some third-party CJK IME on Safari browser, 
						the characters are entered at the keydown state even if 
						they are not keyup.
						*/
						const p = ps.textContent || '';
						ps.textContent = p.substring(0, p.length - 1);
					}
					Caret.moveEndIn(span);

					return false;
				}
			}
		},
		keyUp = (e: KeyboardEvent) => {
			if (keyUpDebt !== null) {
				keyUpDebt(e);
				keyUpDebt = null;
				e.preventDefault();
			}
		};

	onMount(() => {
		document.execCommand('defaultParagraphSeparator', false, 'p');
	});
</script>

<div class="input" on:keydown={keyDown} on:keyup={keyUp} contenteditable bind:innerHTML={value} />

<style>
	div {
		width: 100%;
		border: 3px solid #000;
		box-sizing: border-box;
		min-height: 80px;
		background: none;
		padding: 1em;
		margin: 0;
		font-size: inherit;
		color: inherit;
	}
	div:focus {
		outline: none;
	}
	:global(div.input span) {
		padding: 0 5px;
		color: #03f;
		font-weight: bold;
	}
</style>
