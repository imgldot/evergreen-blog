---
title: "Dynamically generate Open Graph Images using Cloudinary"
category: ''
pubDate: 'May 18 2024 23:59'
---

We can use Cloudinary, an image hosting service and CDN, to dynamically generate open graph images for each blog posts.

For example, below link will generate the following open graph image:

```text
https://res.cloudinary.com/dtxawyaxa/image/upload/w_1600,h_836,q_100/l_text:Roboto_48:%253E%2524%2520PENSIEVE,co_rgb:ffe4e699,c_fit,w_1300/fl_layer_apply,g_north_west,x_100,y_100/l_text:Roboto_72_bold:Transitioning%2520from%2520Monitors%2520to%2520a%2520VR%2520Workspace,co_rgb:ffe4e6,c_fit,w_1300,h_240/fl_layer_apply,g_south_west,x_100,y_190/l_text:Roboto_48:jiiyoo.me%25E3%2583%25BBDec%252028%252C%25202022%25E3%2583%25BB%253Epublished,co_rgb:ffe4e680,c_fit,w_1400/fl_layer_apply,g_south_west,x_100,y_100/base-layer.png
```

![](/images/dynamically_generate_open_graph_images_using_cloudinary/img1.webp)

## Prep

We need a couple of preparations before we start.

First, we need to create an account on [Cloudinary](https://cloudinary.com/). You can easily sign up using your Google or GitHub account. 

Next, we need a base layer which we'll use to generate our Open graph (OG) Images. We'll use this image, `base-layer.png`. 

![](/images/dynamically_generate_open_graph_images_using_cloudinary/img2.webp)

Log in to the cloudinary account and upload your base image. Ensure to modify the public ID of the image since its name will be utilized in the code.

## Creating Layers

We're going to create a utility function called `createOGImage` to generate a URL for our customized, styled, and formatted layers.

### Base Layer

Replace `<cloud name>` with your cloud name which can be found in the dashboard, and if the public ID of the image differs, substitute `base-layer.png` with the correct public ID.

```js
export const createOGImage = ({title, meta}) => {
	[
		// cloud name can be found on the dashboard
		'https://res.cloudinary.com/<cloud name>/image/upload',
		
		// width, height, quality of an image
		'w_1600,h_836,q_100',
		
		// public ID of the base layer image with extension
		'base-layer.png'
	].join('/')
}
```

To set the width and height of the image, use the `w_` and `h_` flags. In this example, it will set the dimensions to 1600x836.

Additionally, `q_100` indicates that we want the best visual quality for the image. The reason why we add this flag is because Cloudinary automatically compresses images to reduce size, but for our OG Image, we want to disable this default behavior.
### Text Layer

Now, let's add a text layer on top of our base layer.

We'll utilize `l_text:<font>_<size>` to specify the font family and size, `co_rgb:<hex>` for color, and `g_` (gravity) for positioning the text.

```js
export const createOGImage = ({title, meta}) => {
	[
		'https://res.cloudinary.com/<cloud name>/image/upload',
		'w_1600,h_836,q_100',
	
		// TITLE
		`l_text:Roboto_72_bold:${e(title)},
		co_rgb:ffe4e6,c_fit,w_1300,h_240`,
		
		// Positioning
		`fl_layer_apply,g_south_west,x_100,y_190`,

		'base-layer.png'
	].join('/')
}

const e = (str: string) => encodeURIComponent(encodeURIComponent(str))
```

Here are the flags we used to position the title:
- `l_text:Roboto_72_bold` - Utilizes the Roboto font with a size of 72px and makes it bold.
- `${e(title)}` - This is where we insert the post title. We use `encodeURIComponent` to escape characters like quotes, question marks, etc.
- `co_rgb:ffe4e6` - Sets the font color as `#ffe4e6`.
- `c_fit` - Sets the crop mode to `fit`, allowing longer text to wrap to multiple lines.
- `w_1300,h_240` - Limits the width and height of the text.

For positioning the text:

- `fl_layer_apply` - Denotes the end of the definition.
- `g_south_west` - Positions the text to the south (bottom) and west (left).
- `x_100,y_190` - Offsets the layer (x, y) from the point of gravity.

![](/images/dynamically_generate_open_graph_images_using_cloudinary/img3.webp)

### Multiple Texts

We can repeat the same process above to add more texts to the layer.

```js
export const createOGImage = ({title, meta}) => {
	[
		'https://res.cloudinary.com/<cloud name>/image/upload',
		'w_1600,h_836,q_100',

		// TITLE
		`l_text:Roboto_72_bold:${e(title)},
		co_rgb:ffe4e6,c_fit,w_1300,h_240`,
		// Positioning
		`fl_layer_apply,g_south_west,x_100,y_190`,

	    // META
	    `l_text:Roboto_48:${e(meta)},co_rgb:ffe4e680,c_fit,w_1400`,
	    // Positioning
	    `fl_layer_apply,g_south_west,x_100,y_100`,
		
		'base-layer.png'
	].join('/')
}

const e = (str: string) => encodeURIComponent(encodeURIComponent(str))
```

![](/images/dynamically_generate_open_graph_images_using_cloudinary/img4.webp)
## Example

Here is an example of how to call the `createOGImage` function.

```jsx
import { createOGImage } from '/lib/createOGImage'

const Post = (post) => {
	const ogImage = createOGImage({
		post.title,
		meta: [
			post.date,
			...post.tags
		].join('・')
	})

	return (
		<Head>
        <meta property="og:image" content={ogImage} />
        <meta property="og:image:width" content="1600" />
        <meta property="og:image:height" content="836" />
        <meta property="og:image:alt" content={title} />
        <meta property="twitter:description" content={title} />
        <meta property="twitter:card" content="summary_large_image" />
		</Head>
	)
}
```


## References
- DELBA | Open Graph Images: Automatically generate OG images from post content. (n.d.). https://delba.dev/blog/next-blog-generate-og-image#what-is-cloudinary