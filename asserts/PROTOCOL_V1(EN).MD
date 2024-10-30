# PIPER SDK Protocol V1 Documentation

## Protocol V1 Analysis

The entire protocol analysis is divided into two layers: one is the base class, and the other is the actual data parsing class.

### C_PiperParserBase Base Class

This code defines the `C_PiperParserBase` class, which serves as a base class for parsing protocols related to robotic arms. It provides a range of functionalities for handling the parsing and conversion of data protocols for robotic arms. Below is a breakdown of the main structure and functions of this class:

#### 1. C_PiperParserBase Class

This is an abstract base class for parsing robotic arm protocol data.
It contains several abstract methods that derived classes must implement to complete specific parsing and encoding operations.

1.1 Inner Class: ProtocolVersion

Defines an enumeration for the protocol version of the robotic arm.
The enumeration values include `ARM_PROTOCOL_UNKNOWN` and `ARM_PROTOCOL_V1`, and derived classes can extend and specify particular protocol versions.

1.2 Abstract Methods

- `DecodeMessage()`: Decodes messages by converting CAN data frames into the specified type. Derived classes must implement this method.
- `EncodeMessage()`: Encodes messages by converting input data into CAN data frames. Derived classes must implement this method.
- `GetParserProtocolVersion()`: Retrieves the currently used protocol version. Derived classes must implement this method.

#### 2. Data Conversion Methods

This class includes various methods for integer type conversions, designed to convert input values into integers or byte lists in specified formats:

2.1 Conversion to Different Bit-Width Integers

- `ConvertToNegative_8bit`, `ConvertToNegative_16bit`, `ConvertToNegative_32bit`: Convert the input integer to a signed or unsigned integer of 8, 16, or 32 bits.
  
- `ConvertToNegative_int8_t`, `ConvertToNegative_uint8_t`: Specific conversion to an 8-bit signed or unsigned integer.

- `ConvertToNegative_int16_t`, `ConvertToNegative_uint16_t`: Specific conversion to a 16-bit signed or unsigned integer.

- `ConvertToNegative_int32_t`, `ConvertToNegative_uint32_t`: Specific conversion to a 32-bit signed or unsigned integer.

2.2 Conversion to Byte Lists

- `ConvertToList_8bit`, `ConvertToList_16bit`, `ConvertToList_32bit`: Convert the input integer to an integer list of 8, 16, or 32 bits.

- `ConvertToList_int8_t`, `ConvertToList_uint8_t`: Specific conversion to a byte list of 8-bit signed or unsigned integers.

- `ConvertToList_int16_t`, `ConvertToList_uint16_t`: Specific conversion to a byte list of 16-bit signed or unsigned integers.

- `ConvertToList_int32_t`, `ConvertToList_uint32_t`: Specific conversion to a byte list of 32-bit signed or unsigned integers.

#### 3. Byte Conversion to Integer

- `ConvertBytesToInt()`: Converts a sequence of bytes into an integer, supporting both big-endian and little-endian alignment.
  
  This function is used to convert the raw CAN data frame (byte type) received into an integer, after which the data type can be defined by calling the conversion functions above.

### C_PiperParserV1 Derived Class

1. Class and Inheritance

   - `C_PiperParserV1` inherits from `C_PiperParserBase` and represents the V1 version of the Piper robotic arm data parsing class.
   - The inheritance structure indicates that this class is likely based on an abstract or more general parsing framework to handle protocol data for robotic arms.

2. Main Functions

   - **Protocol Version Management**: Provides the `GetParserProtocolVersion()` method to return the currently used protocol version.
   - **Message Decoding**: `DecodeMessage()` parses messages transmitted over the CAN bus, converting them into data types defined within the program.
   - **Message Encoding**: `EncodeMessage()` converts program-defined data types into CAN data frames for transmission.

3. Key Methods

   - **GetParserProtocolVersion()**
     - Returns the current protocol version (`ARM_PROTOCOL_V1`), identifying the protocol version adapted by the parser.

   - **DecodeMessage(rx_can_frame, msg)**
     - Primarily used to decode CAN messages.
     - Decodes data frames into specific robotic arm data structures based on the CAN message ID (`can_id`).
     - Utilizes extensive `if-elif` statements to execute decoding logic based on different `can_id` values.
     - Data decoding methods include status feedback, end-effector pose, joint angles, gripper feedback, and driver information (high-speed and low-speed).
     - Involves multiple data conversion methods, such as `ConvertToNegative_*bit()`, to handle byte-to-value conversion while retaining value sign.

   - **EncodeMessage(msg, tx_can_frame)**
     - Primarily used to encode internal data structures into CAN messages.
     - Encodes data based on `msg` type and constructs CAN data frames.
     - Supports encoding of various robotic arm control commands (e.g., motion control, end-effector control, joint control).
     - Calls the `ConvertToList_*bit()` methods during data encoding to convert internal values to the CAN data format.

4. Data Processing Workflow

   - **Data Reception and Decoding**:
     - `rx_can_frame` serves as input, containing the received CAN message.
     - `DecodeMessage()` parses `rx_can_frame`, converting byte data into the `msg` object defined in the program.
   - **Data Transmission and Encoding**:
     - `msg` serves as input, containing control or feedback data to be transmitted.
     - `EncodeMessage()` encodes `msg` data into `tx_can_frame`, preparing it for CAN bus transmission.

5. Extensibility and Maintainability

   - Each CAN message’s parsing and encoding logic is encapsulated in specific conditional branches, making it easier to extend new message types in the future.
   
   - `DecodeMessage` and `EncodeMessage` serve as the main entry points for business logic.
   
   - Extensive use of data conversion methods ensures consistency in byte-to-value conversions.
