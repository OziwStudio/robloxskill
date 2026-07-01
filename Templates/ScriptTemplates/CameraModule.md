```luau

local TweenService = game:GetService("TweenService")

local CameraZoom = {}

CameraZoom.DefaultFOV = 70
CameraZoom.ZoomFOV = 60
CameraZoom.OutFOV = 80
CameraZoom.TweenTime = 0.25
CameraZoom.EasingStyle = Enum.EasingStyle.Sine
CameraZoom.EasingDirection = Enum.EasingDirection.Out

local activeTween: Tween? = nil

function CameraZoom.Set(camera: Camera, targetFOV: number, tweenTime: number?)
	if activeTween then
		activeTween:Cancel()
		activeTween = nil
	end

	local info = TweenInfo.new(
		tweenTime or CameraZoom.TweenTime,
		CameraZoom.EasingStyle,
		CameraZoom.EasingDirection
	)

	activeTween = TweenService:Create(camera, info, {
		FieldOfView = targetFOV,
	})

	activeTween:Play()
	activeTween.Completed:Once(function()
		if activeTween then
			activeTween = nil
		end
	end)

	return activeTween
end

function CameraZoom.ZoomIn(camera: Camera, tweenTime: number?)
	return CameraZoom.Set(camera, CameraZoom.ZoomFOV, tweenTime)
end

function CameraZoom.ZoomOut(camera: Camera, tweenTime: number?)
	return CameraZoom.Set(camera, CameraZoom.OutFOV, tweenTime)
end

return CameraZoom

```