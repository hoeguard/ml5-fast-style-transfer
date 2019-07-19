<template lang="pug">
v-container(grid-list-md text-xs-center)
	v-layout(row wrap)
		v-flex(xs12)
			v-card
				.uniform
					img.image.pa-2(
						:src="image"
					)
				.uniform
					img.image.pa-2(
						:src="stylePreview"
					)
				.uniform
					img#image.image.pa-2(
						:src="imageSrc"
					)
				v-card-actions
					v-spacer
					v-dialog(
						width="500"
						v-model="selectImage"
					)
						template(v-slot:activator="{ on }")
							v-btn(
								dark
								v-on="on"
								@click="startCapture"
							) Take Picture

						v-flex(xs12)
							v-card.pa-4
								canvas#canvas(
									width="320" height="320"
								)
								.video-container
									video#video(
										width="320" height="320"
										autoplay=true
									)
							v-card-actions
								v-spacer
								v-btn(
									color="primary"
									@click="capture"
								) Take Picture
								v-spacer
					v-spacer
					v-dialog(
						width="500"
						v-model="selectStyle"
					)
						template(v-slot:activator="{ on }")
							v-btn(
								dark
								v-on="on"
							) Select Style

						v-flex(xs12)
							v-card
								v-container(grid-list-sm fluid)
									v-layout(row wrap)
										v-flex(
											v-for="style in styles"
											:key="style"
											xs4
										)
											v-hover
												v-card.ma-1(
													tile
													slot-scope="{ hover }"
    												:class="`elevation-${hover ? 12 : 2}`"
													:style="{ cursor: 'pointer'}"
												)
													v-img(
														:src="`/${style}.jpg`"
														aspect-ratio="1"
														class="grey lighten-2"
														@click="setStyle(style)"
													)
					v-spacer
					v-btn(
						color="primary"
						@click="styletransfer"
					) Transfer Style
					v-spacer
</template>

<script>
import ml5 from 'ml5'

const constraints =  {
	"video": {
		width: {
			exact: 320
		}
	}
}

export default {
	data() {
		return {
			style: undefined,
			image: '/puppy.jpg',
			imageSrc: '/puppy.jpg',
			MediaStream: window.MediaStream,
			selectStyle: false,
			selectImage: false,
			model: 'udnie',
			styles: [
				'la_muse',
				'rain_princess',
				'scream',
				'udnie',
				'wave',
				'wreck'
			]
		}
	},
	computed: {
		stylePreview() {
			return `/${this.model}.jpg`
		},
		styleModel() {
			return `/models/${this.model}`
		}
	},
	mounted() {
		this.style = ml5.styleTransfer(this.styleModel, () => {
			console.log('model loaded')
		})
	},
	methods: {
		setStyle(style) {
			this.selectStyle = false
			this.model = style
			
			this.style = ml5.styleTransfer(this.styleModel, () => {
				console.log('model loaded')
			})
		},
		async styletransfer() {
			let image = document.getElementById('image')
	
			await this.style.transfer(image, (err, result) => {
				this.imageSrc = result.src
			})
		},
		startCapture() {
			navigator.mediaDevices.getUserMedia(constraints)
				.then(this.streamCapture)
				.catch(e => console.log(e))
		},
		streamCapture(MediaStream) {
			this.MediaStream = MediaStream
			document.getElementById('video').srcObject = this.MediaStream
		},
		stopCapture() {
			this.MediaStream.getVideoTracks()[0].stop()
			this.selectImage = false
		},
		capture() {
			navigator.mediaDevices.getUserMedia({video: true})
  				.then(this.gotMedia)
				.catch(error => console.error('getUserMedia() error:', error))
			this.stopCapture()
		},
		gotMedia(mediaStream) {
			let imageHeight, imageWidth

			const mediaStreamTrack = mediaStream.getVideoTracks()[0]
			const imageCapture = new ImageCapture(mediaStreamTrack)

			imageCapture.takePhoto()
				.then(blob => {
					this.imageSrc = URL.createObjectURL(blob)

					// const canvas = document.createElement('canvas')
					let canvas = document.getElementById('canvas')
					canvas.width = 320
					canvas.height = 320
					const ctx = canvas.getContext('2d')

					const img = new Image()

					imageCapture.getPhotoSettings()
						.then(ps => {
							// ctx.drawImage(img, 0, 0, canvas.width, canvas.height)

							let ratio = ps.imageWidth / ps.imageHeight
							let width = ps.imageHeight * ratio
							let height = ps.imageWidth / ratio
							let xOffset = 0
							let yOffset = 0

							if (width < canvas.width) {
								let zoom = canvas.width / width
								width = zoom * width
								height = zoom * height
								yOffset = -(height - canvas.height) / 2
							}

							if (height < canvas.height) {
								let zoom = canvas.height / height
								width = zoom * width
								height = zoom * height
								xOffset = -(width - canvas.width) / 2
							}
							
							img.onload = () => {
								ctx.drawImage(img, xOffset, yOffset, width, height)
								let canvasImg = canvas.toDataURL("image/jpeg")
								this.imageSrc = this.image = canvasImg
							}


							img.src = this.imageSrc
						})
						.catch(err => console.error(err))
				})
				.catch(error => console.error('takePhoto() error:', error))
		}
	}
}
</script>

<style lang="styl">
.uniform {
	width: 30%;
	padding-top: 30%;
	position: relative;
	display: inline-block;
}
.image {
	position: absolute;
	height: 100%;
	width: 100%;
	top: 0;
	left: 0;
	bottom: 0;
	right: 0;
}

.video-container, canvas {
	height: 320px;
	width: 320px;
	position: relative;
	overflow: hidden;
	margin: 0 auto;
}

#video {
	position: absolute;
    min-width: 100%;
    min-height: 100%;
    width: auto;
    height: auto;
    top: 0;
    left: 50%;
    transform: translateX(-50%);
}

#canvas {
	display: none;
}
</style>
