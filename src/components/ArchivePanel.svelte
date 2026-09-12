<script lang="ts">
import { onMount, tick } from "svelte";
import {
	type DiscoverPost,
	filterPosts,
	parsePostFilters,
} from "../utils/post-filter";
import { BOOKMARK_EVENT, readBookmarks } from "../utils/preferences";
import { getPostUrlBySlug } from "../utils/url-utils";
import BookmarkButton from "./BookmarkButton.svelte";

export let sortedPosts: DiscoverPost[] = [];
let filters = parsePostFilters(new URLSearchParams());
let bookmarks: string[] = [];
let ready = false;
$: categoryOptions = [
	...new Set(
		sortedPosts
			.map((post) => post.data.category?.trim())
			.filter((value): value is string => !!value),
	),
].sort();
$: tagOptions = [
	...new Set(sortedPosts.flatMap((post) => post.data.tags)),
].sort();
$: filtered = filterPosts(sortedPosts, filters, bookmarks);

async function syncUrl() {
	await tick();
	const address = new URL(window.location.href);
	for (const name of ["q", "category", "tag", "saved", "sort", "uncategorized"])
		address.searchParams.delete(name);
	if (filters.query.trim()) address.searchParams.set("q", filters.query.trim());
	for (const category of filters.categories)
		address.searchParams.append("category", category);
	for (const tag of filters.tags) address.searchParams.append("tag", tag);
	if (filters.saved) address.searchParams.set("saved", "true");
	if (filters.uncategorized) address.searchParams.set("uncategorized", "true");
	if (filters.sort !== "newest") address.searchParams.set("sort", filters.sort);
	window.history.replaceState(window.history.state, "", address);
	window.dispatchEvent(new Event("zebrafish:navigation"));
}
function toggleTag(tag: string) {
	filters.tags = filters.tags.includes(tag)
		? filters.tags.filter((item) => item !== tag)
		: [...filters.tags, tag];
	syncUrl();
}
function reset() {
	filters = parsePostFilters(new URLSearchParams());
	syncUrl();
}
onMount(() => {
	const updateBookmarks = () => {
		bookmarks = readBookmarks();
	};
	const updateFilters = () => {
		filters = parsePostFilters(new URLSearchParams(window.location.search));
	};
	updateFilters();
	updateBookmarks();
	ready = true;
	window.addEventListener(BOOKMARK_EVENT, updateBookmarks);
	window.addEventListener("storage", updateBookmarks);
	window.addEventListener("popstate", updateFilters);
	return () => {
		window.removeEventListener(BOOKMARK_EVENT, updateBookmarks);
		window.removeEventListener("storage", updateBookmarks);
		window.removeEventListener("popstate", updateFilters);
	};
});
function dateLabel(value: Date | string) {
	return new Intl.DateTimeFormat("zh-TW", {
		year: "numeric",
		month: "short",
		day: "numeric",
		timeZone: "UTC",
	}).format(new Date(value));
}
</script>

<section class="card-base archive-panel" aria-labelledby="archive-title">
  <p class="eyebrow">EXPLORE THE ARCHIVE</p>
  <h1 id="archive-title">{filters.saved ? "稍後閱讀" : "文章探索"}</h1>
  <p class="section-description">{filters.saved ? "收藏感興趣的文章，留給下一次靜心閱讀。收藏只儲存在此瀏覽器。" : "從一個關鍵字出發，找到下一篇值得閱讀的文章。"}</p>
  <div class="archive-controls">
    <label class="search-field"><span>搜尋文章</span><input type="search" placeholder="標題、摘要或標籤…" bind:value={filters.query} on:input={syncUrl} disabled={!ready}/></label>
    <label><span>分類</span><select value={filters.uncategorized ? "__uncategorized" : (filters.categories[0] || "")} disabled={!ready} on:change={(event) => { const value = event.currentTarget.value; filters.categories = value && value !== "__uncategorized" ? [value] : []; filters.uncategorized = value === "__uncategorized"; syncUrl(); }}>
      <option value="">全部分類</option>{#each categoryOptions as category}<option value={category}>{category}</option>{/each}<option value="__uncategorized">未分類</option>
    </select></label>
    <label><span>排序</span><select bind:value={filters.sort} on:change={syncUrl} disabled={!ready}><option value="newest">最新優先</option><option value="oldest">最早優先</option><option value="title">依標題</option></select></label>
  </div>
  <div class="tag-filters" aria-label="依標籤篩選，可選多個">
    {#each tagOptions as tag}<button class="filter-chip" aria-pressed={filters.tags.includes(tag)} disabled={!ready} on:click={() => toggleTag(tag)}># {tag}</button>{/each}
  </div>
  <div class="archive-summary">
    <p role="status">找到 <strong>{filtered.length}</strong> 篇文章</p>
    <label class="saved-filter"><input type="checkbox" bind:checked={filters.saved} on:change={syncUrl} disabled={!ready}/> 只看收藏</label>
    <button class="reset-filter" on:click={reset} disabled={!ready}>清除篩選</button>
  </div>
  <noscript><p class="section-description">以下為全部文章；啟用 JavaScript 即可使用篩選與收藏。</p></noscript>
  <div class="archive-results">
    {#each filtered as post (post.slug)}
      <article class="archive-item">
        <div class="archive-item-meta"><time datetime={new Date(post.data.published).toISOString()}>{dateLabel(post.data.published)}</time><span>{post.data.category || "未分類"}</span></div>
        <h2><a href={getPostUrlBySlug(post.slug)}>{post.data.title}<span aria-hidden="true">↗</span></a></h2>
        {#if post.data.description}<p>{post.data.description}</p>{/if}
        <div class="archive-item-bottom"><div class="archive-item-tags">{#each post.data.tags as tag}<button on:click={() => toggleTag(tag)} disabled={!ready}>#{tag}</button>{/each}</div><BookmarkButton slug={post.slug} title={post.data.title}/></div>
      </article>
    {:else}
      <div class="empty-state"><span aria-hidden="true">⌕</span><h2>{filters.saved ? "還沒有符合條件的收藏" : "暫時沒有找到文章"}</h2><p>{filters.saved ? "在文章旁點選「稍後閱讀」，或放寬篩選條件。" : "試試更短的關鍵字，或清除分類與標籤。"}</p><button class="tool-button" on:click={reset}>瀏覽所有文章 →</button></div>
    {/each}
  </div>
</section>
