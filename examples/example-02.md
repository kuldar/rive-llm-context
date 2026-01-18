# TotalConverter
Source: https://rive.app/marketplace/25749-48126-scripting-converter/

## TotalConverter script
```
type TotalConverter = {
  feesVM: Input<Data.Fees>,
}

function init(self: TotalConverter): boolean
  return true
end

function convert(self: TotalConverter, input: DataValueNumber): DataValueNumber
  print('converting')
  local dv: DataValueNumber = DataValue.number()
  print(dv.isString) -- false

  -- Add all of the fees from the fees list.
  for i = 1, self.feesVM.fees.length do
    local item = self.feesVM.fees[i]

    local value = item:getNumber('feesValue')
    if value ~= nil then
      dv.value += value.value
    end
  end

  -- Add the input value (the tip) to the fees.
  dv.value += input.value
  return dv
end

function reverseConvert(
  self: TotalConverter,
  input: DataValueNumber
): DataValueNumber
  -- Since this is a total converter, we don't need to handle target-to-source.
  -- Instead we'll just return the input value.
  return input
end

return function(): Converter<TotalConverter, DataValueNumber, DataValueNumber>
  return {
    convert = convert,
    reverseConvert = reverseConvert,
    init = init,
    feesVM = late(),
  }
end
```
