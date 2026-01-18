# Slot Machine
Source: https://rive.app/marketplace/25759-48234-slot-machine-game-with-scripting/

## SlotMachine script
```
local Utils = require('SlotMachineUtils')

-- Define the script's data and inputs.
type SlotMachine = {
  scoreLeftTotal: Input<number>,
  scoreCenterTotal: Input<number>,
  scoreRightTotal: Input<number>,
  scoreStarTotal: Input<number>,
  scoreIconLeft: Input<Data.ScoreItem>,
  scoreIconCenter: Input<Data.ScoreItem>,
  scoreIconRight: Input<Data.ScoreItem>,
  chicken: Input<Data.Chicken>,
  spinButton: Input<Data.SpinButton>,
  optionsCount: Input<number>,
  spinnerPosition: { number },
  rotationsPerSpin: Input<number>,
  spinTime: Input<number>,
  longestSpinTime: number,
  wheels: { Data.Wheel },
  wheelLeft: Input<Data.Wheel>,
  wheelCenter: Input<Data.Wheel>,
  wheelRight: Input<Data.Wheel>,
  spinTimer: number,
  starIndex: Input<number>,
  startingSpins: Input<number>,
  starsEarned: number,
}

function initializeGame(self: SlotMachine)
  -- Save the wheels for easy reference
  self.wheels = {
    self.wheelLeft,
    self.wheelCenter,
    self.wheelRight,
  }

  -- Set the spin times for each wheel. Each is a little slower than the last.
  for key, wheel in ipairs(self.wheels) do
    local spinTime = self.spinTime + key * 0.3
    wheel.SpinTime.value = spinTime

    if spinTime > self.longestSpinTime then
      self.longestSpinTime = spinTime
    end
  end

  -- disable spin button
  self.chicken.buttonActive.value = true

  -- Reset values
  self.spinButton.spinsLeft.value = self.startingSpins
  self.scoreIconLeft.FinalValue.value = 0
  self.scoreIconCenter.FinalValue.value = 0
  self.scoreIconRight.FinalValue.value = 0
  self.scoreIconLeft.TotalValue.value = self.scoreLeftTotal
  self.scoreIconCenter.TotalValue.value = self.scoreCenterTotal
  self.scoreIconRight.TotalValue.value = self.scoreRightTotal

  -- Set the initial spin vaue
  spinTo(self, { 0, 1, 3 }, true)
  self.chicken.EnumState.value = 'None'

  startTurn(self)
end

-- On start and when the previous turn ends
function startTurn(self: SlotMachine)
  print('start turn')
  self.chicken.buttonActive.value = true

  -- Reset the win icon values
  for key, wheel in ipairs(self.wheels) do
    wheel.EnumPosition.value = 'None'
  end
end

function handleSpinPressed(self: SlotMachine)
  print('handleSpinPressed')
  local buttonActive = self.chicken.buttonActive

  if buttonActive.value == false then
    print('Button cannot be pressed right now')
    return
  end

  -- disable spin button while spinning
  buttonActive.value = false

  -- take a spin
  self.spinButton.spinsLeft.value -= 1

  -- Get random spin values and initiate the spin
  spinTo(self, {
    math.random(0, self.optionsCount - 1),
    math.random(0, self.optionsCount - 1),
    math.random(0, self.optionsCount - 1),
  })
end

function spinTo(self: SlotMachine, slots: { number }, noCountdown: boolean?)
  -- Initiate the spinning chicken animation
  self.chicken.EnumState.value = 'Spinning'

  local previousValue = -1
  local hasMatch = false

  -- Determine the new spin Y position, which is the current position + the number
  -- of spins + the distance to the correct icon.
  for key, slot in ipairs(slots) do
    local currentSpinPosition = self.spinnerPosition[key]
    local cardPosition = 100 / self.optionsCount * slot
    local finalPosition = Utils.floorTo100(
      currentSpinPosition + self.rotationsPerSpin * 100
    ) + cardPosition

    self.wheels[key].Scroll.value = finalPosition
    self.spinnerPosition[key] = finalPosition

    local currentValue = slot

    if currentValue == previousValue then
      if hasMatch == false then
        print('match on wheel', key - 1, slot)
        setScoreIcon(self, slot, key - 1)
        hasMatch = true
      end
      setScoreIcon(self, slot, key)
      print('match on wheel', key)
    end

    previousValue = currentValue
  end

  -- Start a timer based on the time the spin will take
  if noCountdown then
    return
  end
  self.spinTimer = self.longestSpinTime
end

function setScoreIcon(self: SlotMachine, iconType: number, key: number)
  local wheel = self.wheels[key]
  wheel.iconId.value = iconType

  if iconType == 0 then
    wheel.EnumPosition.value = 'Gem'
    local newScore = self.scoreIconRight.FinalValue.value + 1
    self.scoreIconRight.FinalValue.value =
      math.min(newScore, self.scoreLeftTotal)
  elseif iconType == 1 then
    wheel.EnumPosition.value = 'Heart'
    local newScore = self.scoreIconLeft.FinalValue.value + 1
    self.scoreIconLeft.FinalValue.value =
      math.min(newScore, self.scoreCenterTotal)
  elseif iconType == 2 then
    local newScore = math.min(self.starsEarned + 1, 3)
    if newScore == 1 then
      wheel.EnumPosition.value = 'Star1'
    elseif newScore == 2 then
      wheel.EnumPosition.value = 'Star2'
    elseif newScore == 3 then
      wheel.EnumPosition.value = 'Star3'
    end
    self.starsEarned = newScore
  elseif iconType == 3 then
    wheel.EnumPosition.value = 'Egg'
    local newScore = self.scoreIconCenter.FinalValue.value + 1
    self.scoreIconCenter.FinalValue.value =
      math.min(newScore, self.scoreRightTotal)
  end
end

function onSpinComplete(self: SlotMachine)
  print('onSpinComplete')
  -- Update the chicken's aniimation state
  self.chicken.EnumState.value = 'React'
end
-- Called once when the script initializes.
function init(self: SlotMachine, context: Context): boolean
  initializeGame(self)

  -- Listen for triggers in the view model
  self.spinButton.Spin:addListener(self, handleSpinPressed)
  self.chicken.FinishHappy:addListener(self, startTurn)
  self.chicken.FinishSad:addListener(self, startTurn)

  return true
end

-- Called every frame to advance the simulation.
-- 'seconds' is the elapsed time since the previous frame.
function advance(self: SlotMachine, seconds: number): boolean
  -- If there's a timer, wait then call onSpinComplete
  if self.spinTimer > 0 then
    self.spinTimer -= seconds
    if self.spinTimer <= 0 then
      self.spinTimer = 0
      onSpinComplete(self)
    end
  end
    return true
end

-- Called when any input value changes.
function update(self: SlotMachine) end

-- Called every frame (after advance) to render the content.
function draw(self: SlotMachine, renderer: Renderer) end

-- Return a factory function that Rive uses to build the Node instance.
return function(): Node<SlotMachine>
  return {
    init = init,
    advance = advance,
    update = update,
    draw = draw,
    scoreLeftTotal = 10,
    scoreCenterTotal = 10,
    scoreRightTotal = 10,
    scoreIconLeft = late(),
    scoreIconCenter = late(),
    scoreIconRight = late(),
    chicken = late(),
    spinButton = late(),
    optionsCount = 4,
    spinnerPosition = { 0, 0, 0 },
    rotationsPerSpin = 10,
    wheelLeft = late(),
    wheelCenter = late(),
    wheelRight = late(),
    wheels = {},
    spinTime = 3,
    longestSpinTime = 2.9,
    spinTimer = 0,
    starIndex = 0,
    startingSpins = 5,
    starsEarned = 0,
    scoreStarTotal = 3,
  }
end
```

## SlotMachineUtils script
```
function floorTo100(num: number): number
  local scale = 100
  return math.floor(num / scale) * scale
end

function getWheelValue(num: number, optionsCount: number): number
  local remainder = num % 100
  local base = 100 / optionsCount
  return remainder / base + 1
end

return {
  floorTo100 = floorTo100,
  getWheelValue = getWheelValue,
}
```

## Antenna script
```
-- Define the script's data and inputs.
type Antenna = {
  star1: Input<Data.StarContainer>,
  star2: Input<Data.StarContainer>,
  star3: Input<Data.StarContainer>,
  stars: { Data.StarContainer },
  starsIdleRotations: { number },
  starsX: Input<number>,
  starsY: Input<number>,
  starsRotation: Input<number>,
  bounciness: Input<number>,

  -- Physics state
  prevStarsX: number,
  prevStarsY: number,
  prevStarsRotation: number,
  velocities: { number },
  angularOffsets: { number },
  settleFrames: number,
}

-- Spring physics constants
local SPRING_STIFFNESS = 12 -- How quickly it returns to idle
local DAMPING = 4 -- How quickly oscillations die down

-- Influence multipliers for each motion type
local Y_VELOCITY_INFLUENCE = 0.15 -- Vertical bobbing
local X_VELOCITY_INFLUENCE = 0.08 -- Horizontal movement
local ROTATION_VELOCITY_INFLUENCE = 0.5 -- Head rotation
local GRAVITY_INFLUENCE = 0.3 -- How much head tilt affects antenna droop

-- Head rotation offset (π/2 radians = upright)
local HEAD_NEUTRAL_ROTATION = math.pi / 2

-- Antenna position factors (how much each antenna is affected by lateral motion)
-- Antenna on the left side of head will react differently to rightward motion than antenna on the right
local ANTENNA_LATERAL_BIAS: { number } = { -0.8, 0, 0.8 } -- Left, center, right positioning

-- Called once when the script initializes.
function init(self: Antenna, context: Context): boolean
  self.stars = {
    self.star1,
    self.star2,
    self.star3,
  }

  self.starsIdleRotations = {
    self.star1.StarPathRotation.value,
    self.star2.StarPathRotation.value,
    self.star3.StarPathRotation.value,
  }

  -- Initialize physics state
  self.prevStarsX = self.starsX
  self.prevStarsY = self.starsY
  self.prevStarsRotation = self.starsRotation
  self.velocities = { 0, 0, 0 }
  self.angularOffsets = { 0, 0, 0 }
  self.settleFrames = 2 -- Skip first couple frames to avoid startup bounce

  return true
end

-- Called every frame to advance the simulation.
-- 'seconds' is the elapsed time since the previous frame.
function advance(self: Antenna, seconds: number): boolean
  local dt = math.max(seconds, 0.001)

  -- Let the simulation settle for a couple frames to avoid startup bounce
  if self.settleFrames > 0 then
    self.settleFrames = self.settleFrames - 1
    self.prevStarsX = self.starsX
    self.prevStarsY = self.starsY
    self.prevStarsRotation = self.starsRotation
    return true
  end

  if self.bounciness <= 0 then
    -- No bounciness, reset to idle positions
    for i, star in self.stars do
      star.StarPathRotation.value = self.starsIdleRotations[i]
    end
    self.prevStarsX = self.starsX
    self.prevStarsY = self.starsY
    self.prevStarsRotation = self.starsRotation
    return true
  end

  -- Calculate velocities for all head motions
  local xVelocity = (self.starsX - self.prevStarsX) / dt
  local yVelocity = (self.starsY - self.prevStarsY) / dt
  local rotationVelocity = (self.starsRotation - self.prevStarsRotation) / dt

  -- Store current values for next frame
  self.prevStarsX = self.starsX
  self.prevStarsY = self.starsY
  self.prevStarsRotation = self.starsRotation

  -- Scale physics by bounciness
  local scaledStiffness = SPRING_STIFFNESS / (0.5 + self.bounciness * 0.5)
  local scaledDamping = DAMPING / (0.5 + self.bounciness * 0.5)

  -- Calculate head tilt from neutral (already in radians)
  -- Positive tilt = head tilted right, negative = head tilted left
  local headTiltRadians = self.starsRotation - HEAD_NEUTRAL_ROTATION

  -- Update each antenna with independent spring physics
  for i, star in self.stars do
    local lateralBias = ANTENNA_LATERAL_BIAS[i]

    -- Calculate external forces from head motion
    -- Y velocity: all antenna react similarly (lag behind vertical motion)
    local yForce = -yVelocity * Y_VELOCITY_INFLUENCE * self.bounciness

    -- X velocity: antenna react based on their lateral position
    -- When head moves right, left antenna swings more (it's being pulled), right antenna swings less
    local xForce = xVelocity
      * X_VELOCITY_INFLUENCE
      * self.bounciness
      * (1 - lateralBias)

    -- Rotation velocity: antenna lag behind head rotation
    -- Antenna on the outside of the rotation arc swing more
    local rotForce = -rotationVelocity
      * ROTATION_VELOCITY_INFLUENCE
      * self.bounciness
      * (1 + math.abs(lateralBias) * 0.5)

    -- Gravity effect: when head is tilted, antenna droop in the direction of tilt
    -- sin(tilt) gives us a smooth -1 to 1 value based on head angle
    local gravityForce = math.sin(headTiltRadians)
      * GRAVITY_INFLUENCE
      * self.bounciness

    local externalForce = yForce + xForce + rotForce

    -- Spring force pulls back to idle, but offset by gravity effect
    -- This makes antenna naturally droop toward the direction the head is tilted
    local springTarget = gravityForce
    local springForce = -scaledStiffness
      * (self.angularOffsets[i] - springTarget)

    -- Damping force slows down the motion
    local dampingForce = -scaledDamping * self.velocities[i]

    -- Update velocity and position using spring physics
    local acceleration = springForce + dampingForce + externalForce
    self.velocities[i] = self.velocities[i] + acceleration * seconds
    self.angularOffsets[i] = self.angularOffsets[i]
      + self.velocities[i] * seconds

    -- Apply the offset to the idle rotation
    star.StarPathRotation.value = self.starsIdleRotations[i]
      + self.angularOffsets[i]
  end

  return true
end

-- Called when any input value changes.
function update(self: Antenna) end

-- Called every frame (after advance) to render the content.
function draw(self: Antenna, renderer: Renderer) end

-- Return a factory function that Rive uses to build the Node instance.
return function(): Node<Antenna>
  return {
    init = init,
    advance = advance,
    update = update,
    draw = draw,
    star1 = late(),
    star2 = late(),
    star3 = late(),
    stars = {},
    starsIdleRotations = {},
    starsRotation = 0,
    starsX = 0,
    starsY = 0,
    bounciness = 0.5,
    prevStarsX = 0,
    prevStarsY = 0,
    prevStarsRotation = 0,
    velocities = {},
    angularOffsets = {},
    settleFrames = 0,
  }
end
```
