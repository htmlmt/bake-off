<script context="module">
export async function preload({ params, query }) {
    const res = await this.fetch(`_posts/${params.slug}.md`);
    
    if (res.status === 200) {
        return { postMd: await res.text() };
    } else {
        this.error(res.status, data.message);
    }
}
</script>

<script>
import fm from 'front-matter';
import MarkdownIt from 'markdown-it';

export let postMd;

const md = new MarkdownIt();

$: frontMatter = fm(postMd);
$: post = {
    ...frontMatter.attributes,
    html: md.render(frontMatter.body)
};
</script>

<style>
	.content :global(h2) {
		font-size: 1.4em;
		font-weight: 500;
	}

	.content :global(pre) {
		background-color: #f9f9f9;
		box-shadow: inset 1px 1px 5px rgba(0,0,0,0.05);
		padding: 0.5em;
		border-radius: 2px;
		overflow-x: auto;
	}

	.content :global(pre) :global(code) {
		background-color: transparent;
		padding: 0;
	}

	.content :global(ul) {
		line-height: 1.5;
	}

	.content :global(li) {
		margin: 0 0 0.5em 0;
	}
</style>

<svelte:head>
	<title>{post.title}</title>
</svelte:head>

<div class='
    content
    bg-white
    max-w-6xl
    mx-auto
    px-12
    py-8
'>
    <h1 class='
        font-display
        text-4xl
        mb-4
        leading-tight
        tracking-wider
    '>{post.title}</h1>
    
	{@html post.html}
</div>